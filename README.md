# Node-RED

http://nodered.org

[![Build Status](https://travis-ci.org/node-red/node-red.svg?branch=master)](https://travis-ci.org/node-red/node-red)
[![Coverage Status](https://coveralls.io/repos/node-red/node-red/badge.svg?branch=master)](https://coveralls.io/r/node-red/node-red?branch=master)

Low-code programming for event-driven applications.

![Node-RED: Low-code programming for event-driven applications](http://nodered.org/images/node-red-screenshot.png)

## Quick Start

Check out http://nodered.org/docs/getting-started/ for full instructions on getting
started.

1. `sudo npm install -g --unsafe-perm node-red`
2. `node-red`
3. Open <http://localhost:1880>

## Getting Help

More documentation can be found [here](http://nodered.org/docs).

For further help, or general discussion, please use the [Node-RED Forum](https://discourse.nodered.org) or [slack team](https://nodered.org/slack).

## Developers

If you want to run the latest code from git, here's how to get started:

1.  Clone the code:

        git clone https://github.com/WhiteMatrixTech/node-red-demo
        cd node-red-demo

2.  Install the node-red dependencies

        npm install

3.  Build the code

        npm run build

4.  Run

        npm start

## Deployment

```sh
npm run build

# upload code to server and start server via pm2
npm run start
```

## Contributing

Before raising a pull-request, please read our
[contributing guide](https://github.com/node-red/node-red/blob/master/CONTRIBUTING.md).

This project adheres to the [Contributor Covenant 1.4](http://contributor-covenant.org/version/1/4/).
By participating, you are expected to uphold this code. Please report unacceptable
behavior to any of the project's core team at team@nodered.org.

## Authors

Node-RED is a project of the [OpenJS Foundation](http://openjsf.org).

It is maintained by:

- Nick O'Leary [@knolleary](http://twitter.com/knolleary)
- Dave Conway-Jones [@ceejay](http://twitter.com/ceejay)
- And many others...

## Copyright and license

Copyright OpenJS Foundation and other contributors, https://openjsf.org under [the Apache 2.0 license](LICENSE).

## 关于数据流的传递
### function节点（所有交互都在node端执行）
1. node的input事件监听获取上游节点数据，内部根据processMessage的state继续传递
```
        node.on('input', function(msg, send, done) {
                // msg里获取的是上游节点传入的
        })
        else if(state === RESOLVED) {
                // 这里的msg依然是上游节点传入的
                processMessage(msg, send, done);
        }
```
2. processMessage方法处理，context拿到了当前function节点代码编辑界面的修改后的值
```
        processMessage = function (msg, send, done) {
                    var start = process.hrtime();
                    context.msg = msg;
                    context.__send__ = send;
                    context.__done__ = done;

                    node.script.runInContext(context);
                    context.results.then(function(results) {
                        // results是被function节点编辑后的数据
                        sendResults(node,send,msg._msgid,results,false);
                        if (handleNodeDoneCall) {
                            done();
                        }
                    }
```
3. sendResults方法传给下游节点，内置了send方法

### debug节点
1. input事件监听上游节点数据，传入prepareValue方法
```
this.on("input", function(msg, send, done) {
        // 这里就是上游传入的数据
}
```
2. prepareValue方法获取到数据，截取特定的property，形成output
```
function prepareValue(msg, done) {
        if (preparedEditExpression) {
                // 如果设定了特定的output字段，在这里处理
        } else {
                // 默认处理payload字段
                var property = "payload";
                var output = msg[property];
                // done完成，输出
                done(null,{id:node.id, z:node.z, _alias: node._alias,  path:node._flow.path, name:node.name, topic:msg.topic, property:property, msg:output});
            }
        }
```
