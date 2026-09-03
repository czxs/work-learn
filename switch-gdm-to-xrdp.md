# 1) 安装 xrdp 及其 Xorg 后端
  sudo apt update
  sudo apt install -y xrdp xorgxrdp

  # 2) 允许 xrdp 读取 TLS 私钥
  sudo usermod -aG ssl-cert xrdp

  # 3) 为目标登录用户配置 GNOME 会话
  sudo tee /home/mark/.xsession >/dev/null <<'EOF'
  #!/bin/sh
  export GNOME_SHELL_SESSION_MODE=ubuntu
  export XDG_CURRENT_DESKTOP=ubuntu:GNOME
  export XDG_CONFIG_DIRS=/etc/xdg/xdg-ubuntu:/etc/xdg
  exec gnome-session --session=ubuntu
  EOF

  sudo chown mark:mark /home/mark/.xsession
  sudo chmod 700 /home/mark/.xsession

  # 4) 启用并立即启动服务
  sudo systemctl enable --now xrdp xrdp-sesman

  # 5) 验证
  sudo systemctl is-active xrdp xrdp-sesman
  sudo ss -ltnp | grep ':3389'

  grdctl rdp disable 不是每台新机器都必须执行：

  - 如果没有启用 GNOME 原生 Remote Desktop，直接跳过。
  - 如果已启用 GNOME 原生 RDP，必须先关闭它，否则它和 xrdp 都会争用 3389：

  grdctl rdp disable

  若通过 SSH 以 root/其他用户执行，需要以目标用户 mark 的图形会话执行；最简单可靠的方式是在 GNOME 桌面中通过 Settings → Sharing → Remote Desktop 关闭。

  如果 UFW 已启用，还要只对可信网段开放 RDP：

  sudo ufw allow from <可信网段CIDR> to any port 3389 proto tcp

  不要直接对公网开放 3389。完成后，使用 mark 的 Linux 用户名和密码连接，客户端会话类型选择默认的 Xorg。xrdp 会在用户登录时创建新的 GNOME 桌面，不需要 mark 预先在 GDM/virt-manager 控制台登录。
