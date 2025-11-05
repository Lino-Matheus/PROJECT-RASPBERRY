Projeto Raspberry URL Simples - Kiosk TV Player


🛠️ Configuração do Script do Player
1. Criar e Configurar o Script
  sudo nano /home/senai/start_tv_player.sh

Cole o seguinte o conteudo

  #!/bin/bash
# start_tv_player.sh - Kiosk player para Chromium
# URL padrão: https://172.16.52.16/tv

# Aguardar inicialização do ambiente gráfico e rede
sleep 8

# Configurações do ambiente gráfico
export DISPLAY=:0
export XAUTHORITY=/home/senai/.Xauthority

# Desativar protetor de tela / economia de energia
xset s off
xset -dpms
xset s noblank

# Esconder cursor após 0.5s de inatividade
unclutter -idle 0.5 -root &

# URL do player (página que contém as mídias)
PAGE_URL="https://172.16.52.16/tv"

# Log
LOGFILE="/home/senai/tv-player.log"
echo "$(date): Iniciando TV Player - URL: $PAGE_URL" >> "$LOGFILE"

# Esperar o servidor X estar pronto
while ! xset q > /dev/null 2>&1; do
  sleep 2
done

# Executar Chromium apenas uma vez (sem loop)
echo "$(date): Abrindo Chromium em modo kiosk..." >> "$LOGFILE"
/usr/bin/chromium \
  --kiosk \
  --start-fullscreen \
  --noerrdialogs \
  --disable-infobars \
  --incognito \
  --no-first-run \
  --autoplay-policy=no-user-gesture-required \
  --allow-running-insecure-content \
  --ignore-certificate-errors \
  --disable-gpu-sandbox \
  --disable-software-rasterizer \
  "$PAGE_URL" >> "$LOGFILE" 2>&1 &






2. Definir Permissões do Script

  sudo chown senai:senai /home/senai/start_tv_player.sh
  sudo chmod +x /home/senai/start_tv_player.sh



⚙️ Configuração do Auto Login

sudo raspi-config
sudo raspi-config




4. Criar Diretório de Autostart


  mkdir -p /home/senai/.config/autostart

cole o seguinte conteudi

[Desktop Entry]
Type=Application
Name=TV Player
Exec=/home/senai/start_tv_player.sh
Hidden=false
NoDisplay=false
X-GNOME-Autostart-enabled=true



6. Definir Permissões do Autostart

  sudo chown -R senai:senai /home/senai/.config




🔧 Configuração do Serviço Systemd
7. Criar Serviço Systemd

  sudo nano /etc/systemd/system/tv-player.service


[Unit]
Description=TV Player Kiosk (Chromium)
After=graphical.target network-online.target
Wants=network-online.target

[Service]
Type=simple
User=senai
Group=senai
WorkingDirectory=/home/senai
ExecStart=/bin/bash /home/senai/start_tv_player.sh
Restart=always
RestartSec=5
Environment=DISPLAY=:0
Environment=XAUTHORITY=/home/senai/.Xauthority

[Install]
WantedBy=graphical.target




8. Ativar e Iniciar o Serviço

  sudo systemctl daemon-reload
  sudo systemctl enable tv-player.service



🎮 Comandos de Gerenciamento do Serviço
9. Comandos Úteis para Controle




# Iniciar o serviço
sudo systemctl start tv-player.service

# Parar o serviço
sudo systemctl stop tv-player.service

# Reiniciar o serviço
sudo systemctl restart tv-player.service

# Verificar status do serviço
sudo systemctl status tv-player.service

# Ver logs em tempo real
sudo journalctl -u tv-player.service -f
