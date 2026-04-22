# logs

## Apr 22, 2026

### 哔哩哔哩直播间电池计数器

本项目源于我的另一个仓库的"哔哩哔哩直播间弹幕互动盲盒查询工具" ([boxlive](https://github.com/MrOxOrOxen/yqz/tree/boxlive))。其能够自动统计直播间用户的电池投喂情况（礼物、盲盒、SC、大航海），并在网页端实时展示排名、用户名、总投喂电池数量及礼物详细信息。

共包含以下文件：

- boxlive_v2.py: 监听直播间电池投喂情况并生成json文件
- api_server.py: 后端接口程序，读取json文件并将结果通过HTTP接口暴露给前端
- index.html: 前端程序，HTML5+CSS3+Javascript
- gift_ledger.json: 存储所有用户电池汇总信息的动态数据库
- user_stats.json: 由boxlive_v2.py生成的另一个数据库，与本项目无关

