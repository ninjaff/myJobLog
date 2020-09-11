---
title: swagger-node-express-server
date: 2020-09-11 12:45:18
tags: [swagger, node]
---

### ENV Configer
pkg install node
node -v
npm -v
mkdir node-express-swagger
cd node-express-swagger
npm install express

### Swagger Configer
git clone https://github.com/swagger-api/swagger-ui.git

### Server Configer && Run
cp -r swagger-ui.git/dist static
cat server.js
```
const path = require('path');  
const express = require('express');   
const app = express();  
app.use('/static', express.static(path.join(__dirname, 'static')));  
app.listen(3000, () => console.log('http server start port 3000'));  
```
node server.js



