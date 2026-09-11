## System fonts

System fonts are installed for all users on the system.

Create a directory in the system fonts path and copy the font files there:

```sh
 sudo mkdir -p /usr/local/share/fonts/robofont
 sudo cp ~/Downloads/robofont.ttf /usr/local/share/fonts/robofont/
```

Set permissions and update SELinux labels:

```sh
 sudo chown -R root: /usr/local/share/fonts/robofont
 sudo chmod 644 /usr/local/share/fonts/robofont/*
 sudo restorecon -vFr /usr/local/share/fonts/robofont
```

Update the font cache:

```sh
 sudo fc-cache -v
```