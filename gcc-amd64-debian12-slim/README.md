# gcc amd64 Container

Meant to be ran interactively as a tool container. Basic tools to compile 64-bit C programs are included.

**NOTE**: For SELinux distributions, use the `:z` appendix on the volume mount to make sure SELinux doesn't block you from using the mount.


```bash
podman run --rm -it --volume "$PWD:/workspace:z" --workdir /workspace [CONTAINER NAME]:[TAG]
```
