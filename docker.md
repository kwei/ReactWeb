# 安裝並使用 Docker 的紀錄

## 簡介
```
這裡將會記錄如何從零開始建置起 Docker 並且實際運用，不算是一個教學文但會是我自己的 docs。
```

### 下載並安裝
```
我所使用的電腦版本是 windows 10 但非 windows 10 pro 或 enterprise 版本，所以要透過 Docker Toolbox 來協助安裝在較低版本的作業系統上。
安裝 Docker Toolbox 的流程在官網的 docs 有提到，連結於下方可以自行去了解。
有一個重點是 Docker Toolbox 會啟一個小型 vm，需要額外多做一層 port forwarding。
```
#### [安裝 Docker Toolbox](https://docs.docker.com/toolbox/)
#### [安裝流程](https://docs.docker.com/get-started/)
#### [bash.exe 的路徑問題](https://blog.csdn.net/a632189007/article/details/78601213)

```
安裝好後就是要把想要打包的程式包成 image，首先選擇一個合適的 container。像是我要打包一個 Nodejs base 的程式(react)，那麼我就去 Docker Hub 找看看有沒有我要的 image，而剛好有 Node container。
接著將 image run 起來變成 container 的存在，然後再將我們的程式連同這個 run 起來的環境一起打包成新的 image。
```
```bash
# 將 Node image 下載下來
docker pull node:latest

# 將 Node image 跑起來變成 container，-it 是為了能進到容器裡並能輸入指令
docker run -it node bash

# 查看 container 的一些資訊
cat /etc/*release
```

### Dockerize
```
在想要 dockerize 的專案目錄新增 Dockerfile

├──node_modules
├──public
├────index.html
├──src
├────App.js
├────App.css
├────index.js
├────index.css
├────serviceWorker.js
├────setupTests.js
├──package.json
├──package-lock.json
├──README.md
└──Dockerfile <-- 像這樣
```
```bash
(Dockerfile)
# 選擇要使用的容器環境，這裡選擇 node:latest image
FROM node:latest

# 設定容器內的工作路徑
WORKDIR "/app"

# 為了把程式運行環境弄好，先把所需套件下載，所以優先把 package.json 複製到 /app 內的跟目錄
COPY package.json ./

# 執行 npm install 將套件下載
RUN npm install && npm cache clean --force

# 接下來把剩下需要的程式碼、檔案都一併複製到 ./app 內的跟目錄
COPY . .

# 設定當 image 啟動時要執行的指令，也就是開啟服務的指令
CMD ["npm","start"]
```
```bash
# 接著在專案目錄下 build image (注意: imagename 須都是小寫)
docker image build -t <image name> .

# build 完成後確認一下是否有成功
docker image ls

# 將 image run 起來，-p 3000:8080 的意思是對內 8080 port、對外 3000 port，-d 將程序跑在背景
docker run -p 3000:8080 -d <image name>

# 下指令後會回傳一個 container id，為了看 container 的 log 可以:
docker logs <container ID>

# 確認目前運行中的容器與相關的資訊
docker ps
```
### docker 相關指令
```bash
# 停止容器運行
docker stop <container ID>

# 刪除 dangling images that <none>:<none>
docker rmi --force $(docker images -f "dangling=true" -q)

# 查詢 docker-machine 的資訊與IP
docker-machine ls

# 關閉 docker machine (也就是VM)，名稱預設 default
docker-machine stop default

# 重啟 docker-machine
docker-machine start default
```
#### [Dangling images](https://www.projectatomic.io/blog/2015/07/what-are-docker-none-none-images/)
#### [Docker cmd](https://docs.docker.com/engine/reference/commandline/docker/)

### PUSH
```bash
# 將 image 加上 tag 
docker tag <image name> <docker hub account>/<image name>

# 登入 docker hub
docker login

# 將存好的 image tag push 上去
docker push <docker hub account>/<image name>
```


