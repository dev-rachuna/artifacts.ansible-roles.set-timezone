# <img src=".gitlab/ansible.png" alt="linux" height="30"/> set-timezone

Rola Ansible konfigurująca strefę czasową i locale na systemach Debian/Ubuntu.

---

## Architektura

```mermaid
flowchart TD
    START([Rozpoczęcie roli]) --> DISPATCH["tasks/main.yml<br/>include_tasks: locale_{{ os_family }}.yml"]

    DISPATCH --> LOCALE_DEBIAN["locale_debian.yml"]

    subgraph LOCALE["Konfiguracja Locale"]
        LOCALE_DEBIAN --> APT["Instalacja pakietów<br/><b>apt:</b> tzdata, locales"]
        APT --> LINEINFILE["Odkomentowanie locale<br/>w /etc/locale.gen<br/><b>lineinfile</b>"]
        LINEINFILE --> CHECK_LOCALE["Sprawdzenie dostępnych locale<br/><b>command:</b> locale -a"]
        CHECK_LOCALE --> LOCALE_EXISTS{Locale<br/>istnieje?}
        LOCALE_EXISTS -- Nie --> LOCALE_GEN["Generowanie locale<br/><b>command:</b> locale-gen"]
        LOCALE_EXISTS -- Tak --> COPY_LOCALE
        LOCALE_GEN --> COPY_LOCALE["Zapis /etc/default/locale<br/><b>copy:</b> LANG, LANGUAGE, LC_ALL"]
    end

    subgraph TZ["Konfiguracja Timezone"]
        COPY_LOCALE --> STAT_TIMEDATECTL["Sprawdzenie /usr/bin/timedatectl<br/><b>stat</b>"]
        STAT_TIMEDATECTL --> HAS_TIMEDATECTL{timedatectl<br/>dostępny?}

        HAS_TIMEDATECTL -- Tak --> GET_TZ["Pobranie bieżącej strefy<br/><b>command:</b> timedatectl show"]
        GET_TZ --> TZ_CHANGED{Strefa<br/>się różni?}
        TZ_CHANGED -- Tak --> SET_TZ["Ustawienie strefy<br/><b>command:</b> timedatectl set-timezone"]
        TZ_CHANGED -- Nie --> PERSIST_TZ

        HAS_TIMEDATECTL -- Nie --> SYMLINK["Symlink /etc/localtime<br/>→ /usr/share/zoneinfo/...<br/><b>file: state=link</b>"]

        SET_TZ --> PERSIST_TZ["Zapis /etc/timezone<br/><b>copy</b>"]
        SYMLINK --> PERSIST_TZ
    end

    PERSIST_TZ --> DONE([Koniec roli])

    style START fill:#4a9,stroke:#333,color:#fff
    style DONE fill:#4a9,stroke:#333,color:#fff
    style LOCALE fill:#e8f4fd,stroke:#2196F3
    style TZ fill:#fff3e0,stroke:#FF9800
```

---

## Contributions

Jeśli masz pomysły na ulepszenia, zgłoś problemy, rozwidl repozytorium lub utwórz Merge Request. Wszystkie wkłady są mile widziane!
[Contributions](CONTRIBUTING.md)

---

## License

[Licencja](LICENSE) oparta na zasadach Creative Commons BY-NC-SA 4.0, dostosowana do potrzeb projektu.

---

## Author Information

| ![Maciej Rachuna](https://gitlab.com/uploads/-/system/user/avatar/8161705/avatar.png?width=50) |
|------------------------------------------------------------------------------------------------|
| [Maciej Rachuna](https://gitlab.commrachuna)                                                   |