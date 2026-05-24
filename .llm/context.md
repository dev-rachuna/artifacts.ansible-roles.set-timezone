# Role: set-timezone

## Purpose

Configures timezone and locale on Debian/Ubuntu systems.

## Supported platforms

| OS      | Versions           |
|---------|--------------------|
| Debian  | bullseye, bookworm |
| Ubuntu  | jammy              |

Minimum Ansible version: 2.12

---

## Variables

All variables are defined in `defaults/main.yml` and can be overridden at play level.

| Variable      | Default         | Description                          |
|---------------|-----------------|--------------------------------------|
| `in_timezone` | `Europe/Warsaw` | Timezone identifier (IANA tz format) |
| `in_locale`   | `pl_PL.UTF-8`   | Locale used in `/etc/locale.gen`     |
| `in_language` | `pl_PL:pl`      | Value for `LANGUAGE` in `/etc/default/locale` |
| `in_lc_all`   | `pl_PL.UTF-8`   | Value for `LC_ALL` in `/etc/default/locale`   |

`vars/main.yml` is empty — no internal role variables.

---

## Task flow

### Locale block (Debian/Ubuntu only)

1. Install `tzdata` via `apt`
2. Install `locales` via `apt`
3. Uncomment or add entry in `/etc/locale.gen` using `lineinfile` with regexp that escapes dots in locale name
4. Check if locale directory exists at `/usr/lib/locale/{{ in_locale }}` (stat)
5. Run `locale-gen {{ in_locale }}` only when locale directory is missing
6. Write `/etc/default/locale` with `LANG`, `LANGUAGE`, `LC_ALL`

### Timezone block (all systems)

7. Check if `/usr/bin/timedatectl` exists (stat)
8. If present — set timezone via `timedatectl set-timezone {{ in_timezone }}`
9. Write `/etc/timezone` with timezone value (Debian only)
10. If `timedatectl` is missing — create symlink `/etc/localtime → /usr/share/zoneinfo/{{ in_timezone }}`

---

## Known issues

### Incorrect use of `changed_when: false`

Tasks `timedatectl set-timezone`, `copy /etc/timezone`, and `locale-gen` all have `changed_when: false`.
This hides real changes from Ansible's change report. `changed_when: false` is intended for commands that never modify state (e.g. read-only queries), not to suppress reporting of actual changes.

### Timezone block not guarded by `os_family`

The `timedatectl` check and call run on all OS families, but the surrounding tasks (`/etc/timezone`, symlink fallback) are Debian-only. On RHEL/rpm-based systems, timedatectl would be called but the follow-up persistence steps would be skipped.

### `lineinfile` regexp edge case

```yaml
regexp: "^#?\\s*{{ in_locale | replace('.', '\\\\.') }}\\s"
```

Requires at least one whitespace character after the locale name. May not match if the locale entry has no trailing whitespace (unlikely in standard `/etc/locale.gen` but possible in custom setups).

### Test playbook

- `tests/main.yml` has the play name "Set hostname" instead of "Set timezone" (copy-paste error)
- No assertions — the test only runs the role but does not verify the result

### No handlers

No service restarts are triggered after locale changes. Services that depend on locale (e.g. `sssd`, `sshd`) are not reloaded. This is acceptable for most provisioning scenarios but may require manual intervention if locale-dependent services are running.

---

## Usage example

```yaml
- name: Configure timezone and locale
  hosts: all
  gather_facts: true
  roles:
    - role: set-timezone
      vars:
        in_timezone: Europe/Warsaw
        in_locale: pl_PL.UTF-8
        in_language: pl_PL:pl
        in_lc_all: pl_PL.UTF-8
```

---

## Files

```
set-timezone/
├── defaults/main.yml       # Default variable values
├── handlers/main.yml       # Empty
├── meta/main.yml           # Galaxy metadata, platform support
├── tasks/main.yml          # All task logic
├── tests/main.yml          # Basic smoke test (no assertions)
├── vars/main.yml           # Empty
├── .gitignore
├── CONTRIBUTING.md
├── LICENSE
├── README.md
└── renovate.json
```
