# Contrat IPC — Aegis (`aegis-core`)

Colonne vertébrale du système : tout transite par ces trois familles de messages.
Source de vérité unique, versionnée (`schema_version`). Sérialisation interne Rust
via serde+bincode (socket Unix daemon ↔ crates) ; le bridge WebSocket réexpose en
**JSON** pour l'UI TypeScript. Décision figée — pas de question ouverte.

## 1. Événements (kernel → daemon)

Tout événement porte une enveloppe commune.

```
EventEnvelope {
  schema_version: u16,
  event_id: u128,            // ULID
  ts: u64,                   // ns epoch monotonic
  source: ExecveProbe | Fanotify | MmapLsm | PrivLsm | NetConnect,
  process: ProcessCtx,       // contexte process commun
  payload: EventPayload,
}

ProcessCtx {
  pid, ppid, tgid,
  exe_path, comm, cmdline,
  uid, euid, gid,
  caps_effective: u64,       // bitmask capabilities
  cgroup_id, container_id?,  // null si hôte
}
```

Payloads :

| Variante | Source | Champs clés |
|---|---|---|
| `Exec` | eBPF `bprm_check`/`execve` | interpreter?, script_inline?, from_writable_dir: bool |
| `File` | fanotify | path, op (`OpenExec`\|`Read`\|`Write`\|`Rename`\|`Create`\|`Unlink`), blocking: bool, response_token? |
| `Mmap` | eBPF `security_mmap_file` | prot_wx: bool, backing (`Anon`\|`File`), path? |
| `Priv` | eBPF `security_capset`/setuid | from_uid, to_uid, caps_delta |
| `Net` | eBPF `security_socket_connect` | dst_ip, dst_port, proto, resolved: bool |

`blocking: true` + `response_token` : fanotify attend un verdict avant de laisser
passer l'accès (mode prévention). Le daemon **doit** répondre avant le timeout
kernel sous peine de blocage système — contrainte dure d'implémentation.

## 2. Verdicts (detection → daemon → UI)

```
Verdict {
  schema_version: u16,
  event_id: u128,            // événement déclencheur
  engine: Yara | Behavioral | Ransomware | Heuristic | Fim,
  severity: Info | Low | Medium | High | Critical,
  category: ThreatCategory,  // cf. detection-catalog.md
  mitre: [TechniqueId],      // ex: ["T1059.004", "T1486"]
  confidence: f32,           // 0.0–1.0
  title, detail,
  recommended_action: Action,
}

Action = Log | Notify | Quarantine(path) | Isolate(pid) | Kill(pid)
```

## 3. Commandes (UI → daemon)

```
Command =
  | SetMode { scope: Global | Category(ThreatCategory), mode: Detection | Prevention }
  | ScanOnDemand { path, recursive }
  | ScanMemory { pid }
  | Quarantine { path }
  | Restore { quarantine_id }
  | KillProcess { pid }
  | AddExclusion { kind: Path | Hash | Process, value, reason }
  | RemoveExclusion { id }
  | AckVerdict { event_id }      // feedback faux positif → exclusion suggérée
```

Chaque commande retourne un `CommandResult { ok, error?, data? }`. Le daemon pousse
en continu `Verdict` et `EventEnvelope` (filtrés) vers l'UI via le même canal.

## Garanties

- Versionnement strict : un client de version incompatible est rejeté proprement.
- Le daemon ne fait jamais confiance à l'UI pour un privilège : l'UI ne fait que
  demander, le daemon décide et applique.
- Tout `Verdict` de sévérité ≥ High et toute `Command` mutante sont journalisés
  (pino-équivalent Rust : `tracing`).
