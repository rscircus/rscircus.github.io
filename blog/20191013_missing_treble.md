If somebody cut off the treble in an audio recording, like in this video:

https://www.youtube.com/watch?v=gom6nEvtl3U

you can fix it, by installing

```bash
sudo apt-get install pulseaudio-equalizer
```

(for Ubuntu-ish \*nixes)

and using the "Full Base & Treble" preset. See also infos at [askubuntu](https://askubuntu.com/a/98).

If you wonder how to start it:

```bash
qpaeq
```

... yepp, not very intuitive. Maybe it doesn't start. Then, go to `/etc/pulse/default.pa` and add the following lines:

```bash
load-module module-equalizer-sink
load-module module-dbus-protocol
```

and restart pulse:

```bash
pulseaudio --kill && pulseaudio --start
```
