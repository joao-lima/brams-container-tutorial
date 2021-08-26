
# BRABU

Primeiro faça o `build` da imagem:
```
docker build -t brabu:3.5 .
```

Execução do `brabu` tendo a pasta `data` compartilhada:
```
xhost +local:root
docker run --rm -ti -e DISPLAY=$DISPLAY  -v /tmp/.X11-unix:/tmp/.X11-unix -v $PWD/data:/data brabu:3.5
```
