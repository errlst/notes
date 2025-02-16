- `hyprland.conf` 添加

  ```conf
  env = XMODIFIERS, @im=fcitx
  env = QT_IM_MODULE, fcitx
  ```

- `~/.gtkrc-2.0` 添加

  ```conf
  gtk-im-module="fcitx"
  ```

- `~/.config/gtk-3.0/settings.ini` 和 `~/.config/gtk-4.0/settings.ini` 添加

  ```conf
  [Settings]
  gtk-im-module=fcitx
  ```
