# <img src=".gitlab/ansible.png" alt="linux" height="30"/> set-timezone

Rola Ansible konfigurująca strefę czasową i locale na systemach Debian/Ubuntu.

---

## Architektura

```plantuml
@startuml
skinparam backgroundColor transparent
skinparam defaultTextAlignment center
skinparam activity {
  BackgroundColor #FFFFFF
  BorderColor #333333
  StartColor #44AA99
  EndColor #44AA99
}

start
:tasks/main.yml
include_tasks: locale_{{ os_family }}.yml;
:locale_debian.yml;

partition "Konfiguracja Locale" #E8F4FD {
  :Instalacja pakietów
  **apt:** tzdata, locales;
  :Odkomentowanie locale
  w /etc/locale.gen
  **lineinfile**;
  :Sprawdzenie dostępnych locale
  **command:** locale -a;

  if (Locale istnieje?) then (Nie)
    :Generowanie locale
    **command:** locale-gen;
  else (Tak)
    :Locale jest dostępne;
  endif

  :Zapis /etc/default/locale
  **copy:** LANG, LANGUAGE, LC_ALL;
}

partition "Konfiguracja Timezone" #FFF3E0 {
  :Sprawdzenie /usr/bin/timedatectl
  **stat**;

  if (timedatectl dostępny?) then (Tak)
    :Pobranie bieżącej strefy
    **command:** timedatectl show;

    if (Strefa się różni?) then (Tak)
      :Ustawienie strefy
      **command:** timedatectl set-timezone;
    else (Nie)
      :Strefa bez zmian;
    endif
  else (Nie)
    :Symlink /etc/localtime
    do /usr/share/zoneinfo/...
    **file:** state=link;
  endif

  :Zapis /etc/timezone
  **copy**;
}

stop
@enduml
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
