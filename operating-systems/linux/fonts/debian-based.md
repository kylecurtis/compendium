For a system-wide installation on Debian-based distros:

```sh
sudo mkdir -p /usr/local/share/fonts/robofont
sudo cp ~/Downloads/robofont.ttf /usr/local/share/fonts/robofont/
```

```sh
sudo chown -R root:root /usr/local/share/fonts/robofont
sudo chmod 755 /usr/local/share/fonts/robofont
sudo chmod 644 /usr/local/share/fonts/robofont/*
```

```sh
sudo fc-cache -f -v
```