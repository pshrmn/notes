# run

```bash
docker run <image>
```

Create a container from the image and start the container

## Tags

* `--name <name>` gives the container a name.
* `--expose <host>:<container>` (short `-p`) specifies what container port to bind to what host (where the container is run) port.
* `-d` runs in detached mode (background).