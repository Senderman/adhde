# Подготовка

Установите пакеты:
```bash
pacman -S xdg-user-dirs
```

# xdg-user-dirs

Помимо XDG Base directories, существуют общепринятые директории для личных файлов пользователя. Например, `~/Documents`, `~/Download`, `~/Videos` и другие. Приложения могут их использовать для своих целей (например, браузер будет знать, куда ему скачивать файлы, видеозаписывающее ПО - куда складывать видео и т.д.). Кроме того, некоторые файловые менеджеры отмечают данные директории отдельными иконками, и выводят их в боковой панели. Существует утилита xdg-user-dirs, которая помогает управлять данными директориями.

Создайте 2 файла:
- `~/dotfiles/xdg-user-dirs/.config/user-dirs.locale`
- `~/dotfiles/xdg-user-dirs/.config/user-dirs.dirs`

И выполните `stow xdg-user-dirs`.

Создайте директории:

```bash
mkdir ~/Desktop
mkdir ~/Download
mkdir ~/Templates
mkdir ~/Shared
mkdir ~/Documents
mkdir ~/Music
mkdir ~/Pictures
mkdir ~/Videos
```

В `~/.config/user-dirs.locale` впишите 1 строку:
```bash
en_US
```

`.config/user-dirs.locale` заполните так:

```bash
XDG_DESKTOP_DIR="$HOME/Desktop"
XDG_DOWNLOAD_DIR="$HOME/Download"
XDG_TEMPLATES_DIR="$HOME/Templates"
XDG_PUBLICSHARE_DIR="$HOME/Shared"
XDG_DOCUMENTS_DIR="$HOME/Documents"
XDG_MUSIC_DIR="$HOME/Music"
XDG_PICTURES_DIR="$HOME/Pictures"
XDG_VIDEOS_DIR="$HOME/Videos"
```

Затем, выполните команду `xdg-user-dirs-update`

Проверьте корректность работы:

```bash
$ xdg-user-dir DOWNLOAD 
/home/USERNAME/Download
```

На этом настройка завершена. Подробнее можно почитать тут: [ArchWiki > XDG user directories](https://wiki.archlinux.org/title/XDG_user_directories)
