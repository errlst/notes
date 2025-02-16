1. https://www.gnome-look.org/browse/ 下载光标主题，拷贝到 `~/.local/share/icons`

2. 安装 nwg-look，并设置光标主题。

   ```shell
   yay -S nwg-look
   ```

3. 配置 hyprland.conf。

   ```conf
   env = XCURSOR_SIZE, 48
   env = QT_CURSOR_SIZE, 48
   env = HYPRCURSOR_SIZE, 48
   ```
