How to connect network in wsl with proxy.


---
# Install wsl
If you are biginner, follow this procedure.

https://www.solveyourtech.com/how-to-install-wsl2-on-windows-11-a-step-by-step-guide-for-beginners


---
# On Windows PowerShell (normal user is fine)
```
py -m pip install --user pproxy
py -m pproxy -l http://0.0.0.0:3128 -vv
```
Keep that terminal open (proxy running).

---
# Allow firewall (Admin PowerShell)
```
netsh advfirewall firewall add rule name="WSL pproxy 3128" dir=in action=allow protocol=TCP localport=3128 profile=private
```

---
# In WSL
```
GW=$(ip route | awk '/default/ {print $3; exit}')
export HTTP_PROXY="http://$GW:3128"
export HTTPS_PROXY="$HTTP_PROXY"
curl -I https://www.google.com --max-time 12
```
If this works, add those exports to ~/.bashrc.
If py command not found
Use python:
```
python -m pip install --user pproxy
python -m pproxy -l http://0.0.0.0:3128 -vv
```

---
# If you want Set WSL proxy permanently
Run this code in wsl.
```
cat >> ~/.bashrc <<'EOF'

# >>> wsl-proxy-auto >>>
# Auto-set proxy for WSL via Windows local proxy (:3128)
_GW=$(/usr/sbin/ip route 2>/dev/null | awk '/default/ {print $3; exit}')
if [ -n "$_GW" ] && timeout 1 bash -lc "</dev/tcp/$_GW/3128" >/dev/null 2>&1; then
  export HTTP_PROXY="http://$_GW:3128"
  export HTTPS_PROXY="$HTTP_PROXY"
  export http_proxy="$HTTP_PROXY"
  export https_proxy="$HTTPS_PROXY"
fi
unset _GW
# <<< wsl-proxy-auto <<<
EOF

source ~/.bashrc
```

Then last execute.
```
GW=$(ip route | awk '/default/ {print $3; exit}')
export HTTP_PROXY="http://$GW:3128"
export HTTPS_PROXY="$HTTP_PROXY"

sudo -E apt-get update \
  -o Acquire::http::Proxy="$HTTP_PROXY" \
  -o Acquire::https::Proxy="$HTTPS_PROXY"

sudo -E apt-get install -y python3-venv python3-pip \
  -o Acquire::http::Proxy="$HTTP_PROXY" \
  -o Acquire::https::Proxy="$HTTPS_PROXY"
```


# You can test running in wsl
```
echo "$HTTP_PROXY"
curl -I https://www.google.com --max-time 10
```
