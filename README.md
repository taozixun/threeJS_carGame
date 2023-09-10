启动项目：npm run serve
内网穿透链接：https://mp.weixin.qq.com/s/BypBsqNswLpNixdR4hM-zA
用的是ngrok

本地启动服务后，在E:\App\内网穿透>文件夹，双击exe，执行ngrok http 8080   

注意：vue项目想这么做内网穿透的话，要//在 vue.config.js文件中添加
module.exports = {
  devServer: {
    historyApiFallback: true,
    allowedHosts: "all",
  }
}