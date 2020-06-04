1. Download a clean copy

```sh
git clone https://github.com/thu-spmi/CAT my-cat
```

2. Build the image at project root.

```sh
cd my-cat
docker build --tag cat-toolkit:pytorch-1.5-cuda10.1-cudnn7-devel -f docker/pytorch-1.x/Dockerfile .
docker build --tag cat-toolkit:pytorch-0.4.1-cuda9-cudnn7-devel -f docker/pytorch-0.4.1/Dockerfile .
```
