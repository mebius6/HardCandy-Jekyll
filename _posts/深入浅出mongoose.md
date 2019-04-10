---
layout: post
title:  "MongoDB"
date:   2019-04-08
tags:  js node 工具

color: 'rgb(154,133,255)'

---
# 深入浅出mongoose
## 连接数据库 
### connect mongoose 有两种方式
第一种:

```js
        'use strict';

        const mongoose = require('mongoose');

        mongoose.connect('mongodb://localhost:27017/test');
        const con = mongoose.connection;
        con.on('error', console.error.bind(console, '连接数据库失败'));
        con.once('open',()=>{
            //成功连接
        })
```
第二种:

```js
        var mongoose = require('mongoose');

        db = mongoose.createConnection('localhost', 'test');
        var schema = new mongoose.Schema({ name: String });
        var collectionName = 'kittens';
        var M = db.model('Kitten', schema, collectionName);
        var silence = new M({ name: "Silence"});
        silence.save(function(err){
        });
```

#### mongoose.createConnection()和mongoose.connect()区别
- 首先，我们需要定义一个连接，如果你的app只用到一个数据库，你应该使用 mongoose.connect。如果你还需要连接其他数据库，使用mongoose.createConnection。
所有的 connect and createConnection 使用 mongodb:// URI, or the parameters host, database, port, options.等参数
- 其次,一旦连接，Connection对象会触发open事件，如果你使用 mongoose.connect, Connection对象就是默认的 mongoose.connection，而使用 mongoose.createConnection 返回值就是 Connection.
- Mongoose会缓存所有命令直到连接上数据库，这意味着你不必等待它连接MongoDB再定义 models，执行 queries 等。
#### mongoose open/connected  close/disconnected的区别
- connected：当此连接成功连接到db时发出。可能会在重新连接的情况下多次发出。
- open：在我们连接后发出，并在所有这些连接模型上执行onOpen。
- disconnected：从数据库断开连接后发出。
- close：我们断开连接并在所有这些连接模型上执行onClose后发出。

```js
        var mongoose = require('mongoose'),
            DB_URL = 'mongodb://localhost:27017/mongoosesample';

        /**
         * 连接
         */
        mongoose.connect(DB_URL);

        /**
          * 连接成功
          */
        mongoose.connection.on('connected', function () {    
            console.log('Mongoose connection open to ' + DB_URL);  
        });    

        /**
         * 连接异常
         */
        mongoose.connection.on('error',function (err) {    
            console.log('Mongoose connection error: ' + err);  
        });    
         
        /**
         * 连接断开
         */
        mongoose.connection.on('disconnected', function () {    
            console.log('Mongoose connection disconnected');  
        }); 
```
#### 指定用户连接

```js
        mongoose.connect('mongodb://用户名:密码@127.0.0.1:27017/数据库名称')
```
如果开启鉴权控制，以用户名"u1"，密码"123456"登录'db1'数据库。执行代码后，控制台输出“连接成功”

```js
        var mongoose = require('mongoose');
        mongoose.connect("mongodb://u1:123456@localhost/db1", function(err) {
            if(err){
                console.log('连接失败');
            }else{
                console.log('连接成功');
            }
        });
```
#### 连接多个数据库
如果你的app中要连接多个数据库，只需要设置多个url以,隔开，同时设置mongos为true

```js
        mongoose.connect('urlA,urlB,...', {
           mongos : true 
        })
```

### connect mongodb
```js
     var mongodb = require('mongodb');
      var server = new mongodb.Server('localhost', 27017, { auto_reconnect: true });
      var db = new mongodb.Db('mydb', server, { safe: true });
      db.open(function (err, db) {
        if (!err) {
          console.log('connect');
        } else {
          console.log(err);
        }
      });
```
## 了解mongoose
<table>
<thead>
<tr><th>Oracle</th><th>MongoDB</th><th>Mongoose</th></tr>
</thead>
<tbody>
<tr>
<td>数据库实例(database instance)</td>
<td>MongoDB实例</td>
<td>Mongoose</td>
</tr>
<tr>
<td>模式(schema)</td>
<td>数据库(database)</td>
<td>mongoose</td>
</tr>
<tr>
<td>表(table)</td>
<td>集合(collection)</td>
<td>模板(Schema)+模型(Model)</td>
</tr>
<tr>
<td>行(row)</td>
<td>文档(document)</td>
<td>实例(instance)</td>
</tr>
<tr>
<td>rowid</td>
<td>_id</td>
<td>_id</td>
</tr>
<tr>
<td>Join</td>
<td>DBRef</td>
<td>DBRef</td>
</tr>
</tbody>
</table>

通过上面的阐述,我们大概能知道了在Mongoose里面有哪几个基本概念.

- Schema: 相当于一个数据库的模板. Model可以通过mongoose.model 集成其基本属性内容. 当然也可以选择不继承.
- Model: 基本文档数据的父类,通过集成Schema定义的基本方法和属性得到相关的内容.
- instance: 这就是实实在在的数据了. 通过 new Model()初始化得到.
他们各自间是怎样的关系呢？ 下图可以清晰的说明, 以上3中实际上就是一个继承一个得到最后的数据.

```js
'use strict';

const mongoose = require('mongoose');

mongoose.connect('mongodb://localhost:27017/test');
const con = mongoose.connection;
con.on('error', console.error.bind(console, '连接数据库失败'));
con.once('open',()=>{
    //定义一个schema
    let Schema = mongoose.Schema({
        category:String,
        name:String
    });
    Schema.methods.eat = function(){
        console.log("I've eatten one "+this.name);
    }
    //继承一个schema
    let Model = mongoose.model("fruit",Schema);
    //生成一个document
    let apple = new Model({
        category:'apple',
        name:'apple'
    });
    //存放数据
    apple.save((err,apple)=>{
        if(err) return console.log(err);
        apple.eat();
        //查找数据
        Model.find({name:'apple'},(err,data)=>{
            console.log(data);
        })
    });
})
```

到这里, 实际上, mongoose我们已经就学会了. 剩下就是看一看官方文档的API–CRUD相关操作. 如果,大家觉得意犹未尽的话,可以继续看下面的深入浅出. 而且, 下面会附上实际应用中, mongoose的写法.

## 深入浅出mongoose
这里,我们根据上面的3个概念深入的展开一下.
### Schema
这实际上是,mongoose中最重要的一个theroy(理论). schema 是用来定义 documents的基本字段和集合的. 在mongoose中,提供了Schema的类。 我们可以通过实例化他, 来实现创建 Schema的效果. 而不需要每次调用 mongoose.Schema()这个丑陋的API.

```js
        // from mongoose author
        var mongoose = require('mongoose');
        var Schema = mongoose.Schema;

        var blogSchema = new Schema({
          title:  String,
          author: String,
          body:   String,
          comments: [{ body: String, date: Date }],
          date: { type: Date, default: Date.now },
          hidden: Boolean,
          meta: {
            votes: Number,
            favs:  Number
          }
        });
```
注意 创建Schema对象时，声明字段类型有两种方法，一种是首字母大写的字段类型，另一种是引号包含的小写字段类型

```js
        var mySchema = new Schema({title:String, author:String});
        //或者 
        var mySchema = new Schema({title:'string', author:'string'});
```
#### add()方法
　如果需要在Schema定义后添加其他字段，可以使用add()方法

```js
        var MySchema = new Schema;
        MySchema.add({ name: 'string', color: 'string', price: 'number' });
```
#### Schema中初始化的相关参数
- safe: 用来设置安全模式. 实际上,就是定义入库时数据的写入限制. 比如写入时限等.

```js
        //使用安全模式. 表示在写入操作时,如果发生错误,也需要返回信息.
         var safe = true;
        new Schema({ .. }, { safe: safe });

        // 自定义安全模式. w为写入的大小范围. wtimeout设置写入时限. 如果超出10s则返回error
        var safe = { w: "majority", wtimeout: 10000 };
        new Schema({ .. }, { safe: safe });
```
- toObject: 用来表示在提取数据的时候, 把documents 内容转化为Object内容输出. 一般而言只需要设置getters为true即可.

```js
        schema.set('toObject', { getters: true });
        var M = mongoose.model('Person', schema);
        var m = new M({ name: 'Max Headroom' });
        //实际打印出来的就是一个Object类型
        console.log(m); // { _id: 504e0cd7dd992d9be2f20b6f, name: 'Max Headroom is my name' }
```
- toJSON： 该是和toObject一样的使用. 通常用来把 documents 转化为Object. 但是, 需要显示使用toJSON()方法,否则,不会起作用. 实际上,没什么卵用…
#### Schema设置的字段及其类型
Schema 之所以能够定义documents, 是因为他可以限制你输入的字段及其类型. mongoose支持的基本类型有:

- String      字符串
- Number      数字    
- Date        日期
- Buffer      二进制
- Boolean     布尔值
- Mixed       混合类型
- ObjectId    对象ID    
- Array       数组
其中, Mixed和ObjectId是mongoose中特有的,ObjectId实际上就是**_id**的一个映射. 同样,mongoose里面有着和所有大众数据库一样的东西. 索引 – indexs
#### mongoose 设置索引
这里设置索引分两种,一种设在Schema filed, 另外一种设在 Schema.index 里.

```js
        //在field 设置
        var animalSchema = new Schema({
          name: String,
          type: String,
          tags: { type: [String], index: true } 
        });
        //在Schema.index中设置.
        animalSchema.index({ name: 1, type: -1 }); 

        //1 表示正序, -1 表示逆序
```
实际上,两者效果是一样的. 看每个人的喜好了. 不过推荐直接在Schema level中设置, 这样分开能够增加可读性. 不过,官方给出了一个建议, 因为在创建字段时, 数据库会自动根据自动排序(ensureIndex). 有可能严重拖慢查询或者创建速度,所以一般而言,我们需要将该option 关闭.

```js
        mongoose.connect('mongodb://user:pass@localhost:port/database', { config: { autoIndex: false } });  //真心推荐
        // or  
        mongoose.createConnection('mongodb://user:pass@localhost:port/database', { config: { autoIndex: false } });  //不推荐
        // or
        animalSchema.set('autoIndex', false);  //推荐
        // or
        new Schema({..}, { autoIndex: false }); //懒癌不推荐
```
另外, Schema 另一大特色就是其methods. 我们可以通过定义其methods,访问到实际上的所有内容.

#### 定义Schema.methods【实例方法】

使用的方法很简单,就是使用 .methods即可.

```js
        // 定义一个schema
        var freshSchema = new Schema({ name: String, type: String });

        // 添加一个fn. 
        animalSchema.methods.findSimilarTypes = function (cb) {
          //这里的this指的是具体document上的this
          return this.model('Animal').find({ type: this.type }, cb);
        }
        // 实际上,我们可以通过schema绑定上,数据库操作的所有方法.
        // 该method实际上是绑定在 实例的 doc上的
        
```

```js
        var mongoose = require('mongoose');
        mongoose.connect("mongodb://u1:123456@localhost/db1", function(err) {
            if(!err){
                var schema = new mongoose.Schema({ num:Number, name: String, size: String });        
                schema.methods.findSimilarSizes = function(cb){
                    return this.model('MyModel').find({size:this.size},cb);
                }
                var MyModel = mongoose.model('MyModel', schema);
                var doc1 = new MyModel({ name:'doc1', size: 'small' });
                var doc2 = new MyModel({ name:'doc2', size: 'small' });
                var doc3 = new MyModel({ name:'doc3', size: 'big' });
                doc1.save();
                doc2.save();
                doc3.save();
                setTimeout(function(){
                    doc1.findSimilarSizes(function(err,docs){
                        docs.forEach(function(item,index,arr){
                            //doc1
                            //doc2
                             console.log(item.name)        
                        })
                    })  
                },0)  
            }
        });
```
定义完methods和property之后, 就到了生成Model的阶段了.

### 实例Model

这里同样很简单,只需要 mongoose.model() 即可.

```js
        //生成,model 类. 实际上就相当于我们的一个collection
        //注意 一定要将model()方法的第一个参数和其返回值设置为相同的值，否则会出现不可预知的结果
        var Animal = mongoose.model('Animal', animalSchema);
        var dog = new Animal({ type: 'dog' });
```
但是, 这里有个问题. 我们在Schema.methods.fn 上定义的方法,只能在 new Model() 得到的实例中才能访问. 那如果我们想,直接在Model上调用 相关的查询或者删除呢？

#### 绑定Model方法【静态方法】

同样很简单,使用 statics 即可.

```js
        // 给model添加一个findByName方法
        animalSchema.statics.findByName = function (name, cb) {
          //这里的this 指的就是Model
          return this.find({ name: new RegExp(name, 'i') }, cb);
        }

        var Animal = mongoose.model('Animal', animalSchema);
        Animal.findByName('fido', function (err, animals) {
          console.log(animals);
        });
```
```js
        var mongoose = require('mongoose');
        mongoose.connect("mongodb://u1:123456@localhost/db1", function(err) {
            if(!err){
                var schema = new mongoose.Schema({ num:Number, name: String, size: String });        
                schema.statics.findByName = function(name,cb){
                    return this.find({name: new RegExp(name,'i')},cb);
                }
                var MyModel = mongoose.model('MyModel', schema);
                var doc1 = new MyModel({ name:'doc1', size: 'small' });
                var doc2 = new MyModel({ name:'doc2', size: 'small' });
                var doc3 = new MyModel({ name:'doc3', size: 'big' });
                doc1.save();
                doc2.save();
                doc3.save();
                setTimeout(function(){
                    MyModel.findByName('doc1',function(err,docs){
                        //[ { _id: 5971e68f4f4216605880dca2,name: 'doc1',size: 'small',__v: 0 } ]
                        console.log(docs);
                    })  
                },0)  
            }
        });
```
由上所示，实例方法和静态方法的区别在于，静态方法是通过Schema对象的statics属性给model添加方法，实例方法是通过Schema对象的methods是给document添加方法
#### 【Methods 和 Statics 的区别】
- statics是给model添加方法，
- methods是给实例（instance）添加方法。
methods和statics的区别

```js
//module.exports = mongoose.model(`Article`, ArticleSchema )
//将article的model保存为文件 article.js

const Article = require('../models/article')

// statics
Article.staticFunc ()

//methods
const article = new Article(arguments)
article.methodFunc()
```
#### 实例化文档document
通过对原型Model1使用new方法，实例化出文档document对象

```js
        var mongoose = require('mongoose');
        mongoose.connect("mongodb://u1:123456@localhost/db1", function(err) {
            if(err){
                console.log('连接失败');
            }else{
                console.log('连接成功');
                var schema = new mongoose.Schema({ num:Number, name: String, size: String});
                var MyModel = mongoose.model('MyModel', schema);
                var doc1 = new MyModel({ size: 'small' });
                console.log(doc1.size);//'small'
            }
        });
```

#### 查询方法
通过schema对象的query属性，给model添加查询方法

```js
        var mongoose = require('mongoose');
        mongoose.connect("mongodb://u1:123456@localhost/db1", function(err) {
            if(!err){
                var schema = new mongoose.Schema({ age:Number, name: String});        
                schema.query.byName = function(name){
                    return this.find({name: new RegExp(name)});
                }
                var temp = mongoose.model('temp', schema);   
                temp.find().byName('huo').exec(function(err,docs){
                    //[ { _id: 5971f93be6f98ec60e3dc86c, name: 'huochai', age: 27 },
                    // { _id: 5971f93be6f98ec60e3dc86e, name: 'huo', age: 30 } ]
                    console.log(docs);
                })  

            }           
        });
```
## 文档新增
文档新增有三种方法，一种是save()方法，另一种是使用模型model的create()方法，最后一种是模型model的insertMany()方法
### save()
- 通过new Model1()创建的文档doc1，必须通过save()方法，才能将创建的文档保存到数据库的集合中，集合名称为模型名称的小写复数版

```js
        save(function (err, doc) {})
```
- 回调函数是可选项，第一个参数为err，第二个参数为保存的文档对象

```js
        var mongoose = require('mongoose');
        mongoose.connect("mongodb://u1:123456@localhost/db1", function(err) {
            if(!err){
                var schema = new mongoose.Schema({ num:Number, name: String, size: String });
                var MyModel = mongoose.model('MyModel', schema);
                var doc1 = new MyModel({ size: 'small' });
                doc1.save(function (err,doc) {
                  console.log(doc);  //{ __v: 0, size: 'small', _id: 5970daba61162662b45a24a1 }
                })
            }
        });
```
### create()
使用save()方法，需要先实例化为文档，再使用save()方法保存文档。而create()方法，则直接在模型Model上操作，并且可以同时新增多个文档

```js
    Model.create(doc(s), [callback])
```
新增{name:"xiaowang"}，{name:"xiaoli"}这两个文档

```js
            var mongoose = require('mongoose');
            mongoose.connect("mongodb://u1:123456@localhost/db1", function(err) {
                if(!err){
                    var schema = new mongoose.Schema({ age:Number, name: String});        
                    var temp = mongoose.model('temp', schema);   
                    temp.create({name:"xiaowang"},{name:"xiaoli"},function(err,doc1,doc2){
                        //{ __v: 0, name: 'xiaowang', _id: 59720d83ad8a953f5cd04664 }
                        console.log(doc1); 
                        //{ __v: 0, name: 'xiaoli', _id: 59720d83ad8a953f5cd04665 }
                        console.log(doc2); 
                    });       
                }           
            });
```
### insertMany()

```js
        Model.insertMany(doc(s), [options], [callback])
```
新增{name:"a"}，{name:"b"}这两个文档

```js
        var mongoose = require('mongoose');
        mongoose.connect("mongodb://u1:123456@localhost/db1", function(err) {
            if(!err){
                var schema = new mongoose.Schema({ age:Number, name: String});        
                var temp = mongoose.model('temp', schema);   
                temp.insertMany([{name:"a"},{name:"b"}],function(err,docs){
                    //[ { __v: 0, name: 'a', _id: 59720ea1bbf5792af824b30c },
                    //{ __v: 0, name: 'b', _id: 59720ea1bbf5792af824b30d } ]
                    console.log(docs); 
                });       
            }
        });
```
## mongoose条件操作符
条件操作符 | 说明
---    | ---
$or  | 或关系
$nor  | 或关系取反
$gt  | 大于
$gte  | 大于等于
$lt  |  小于
$lte  |  小于等于
$ne  | 不等于
$in  |  在多个值范围内
$nin    |   不在多个值范围内
$all   |  匹配数组中多个值
$regex  |  正则，用于模糊查询
$size  | 匹配数组大小
$maxDistance  | 范围查询，距离（基于LBS）
$mod  |  取模运算
$near  |  邻域查询，查询附近的位置（基于LBS）
$exists  |  字段是否存在
$elemMatch  |  匹配内数组内的元素
$within  |  范围查询（基于LBS）
$box  |   范围查询，矩形范围（基于LBS）
$center  |  范围醒询，圆形范围（基于LBS）
$centerSphere  |  范围查询，球形范围（基于LBS）
$slice  |  查询字段集合中的元素（比如从第几个之后，第N到第M个元素）

### mongoose多条件模糊查询
主要用到了query.$or和query.$regex这两个find参数。

- query.$or用于实现多条件查询，其值是一个数组。

```js
        query.or([{ color: 'red' }, { status: 'emergency' }])
```

- query.$regex用于实现模糊查询。

参数介绍： 
Option | Description
---    | ---
参数 i | 加了这个参数，表示不区分大小写
参数 m |  个人理解这个参数是用来匹配value中有换行符(\n)的情形。还有一个情形是：匹配规则中使用了锚,所谓的锚就是^ 开头, $ 结束
参数 s | 允许点字符（.）匹配所有的字符，包括换行符。
参数 x | 官网的大意是忽视空白字符。

现在有以下集合（官网的例子）：

```js
        { "_id" : 100, "sku" : "abc123", "description" : "Single line description." }
        { "_id" : 101, "sku" : "abc789", "description" : "First line\nSecond line" }
        { "_id" : 102, "sku" : "xyz456", "description" : "Many spaces before     line" }
        { "_id" : 103, "sku" : "xyz789", "description" : "Multiple\nline description" }
        var mongoose = require('mongoose');
        mongoose.connect("mongodb://u1:123456@localhost/db1", function(err) {
            if(!err){
                var schema = new mongoose.Schema({ sku:String, description: String});        
                var temp = mongoose.model('temp', schema);
                /*******************************参数i*********************************/
                temp.find( { sku: /adC/i } );//等效于下面这种写法 
                temp.find( { sku: { $regex: 'abC', $options: 'i' } } ); 
                
                 /*******************************参数m*********************************/
                // 匹配description字段的value值中，以大写S开头的value值
                //可以看出，第二条记录中descriptio的值包含\n换行字符，而他之所以能匹配出来就是因为添加了m 参数。 
                temp.find( { description: { $regex: /^S/, $options: 'm' } } ) 
                /** 
                    结果为:
                    { "_id" : 100, "sku" : "abc123", "description" : "Single line description." }
                    { "_id" : 101, "sku" : "abc789", "description" : "First line\nSecond line" } 
                **/
                    
               // 没有添加m参数
               temp.find( { description: { $regex: /^S/} } 
               //结果为: { "_id" : 100, "sku" : "abc123", "description" : "Single line description." }
               
               //锚都不写也就是去掉^符合
               temp.find( { description: { $regex: /S/ } } )
               /**
                   { "_id" : 100, "sku" : "abc123", "description" : "Single line description." }
                   { "_id" : 101, "sku" : "abc789", "description" : "First line\nSecond line" }
               **/
               /**
                 此时可以分析出m参数的使用场景： 
                    应该是为了匹配字段value值中以某个字符开头(^)，或者是某个字符结束($).即便value中包含换行符(\n)也能匹配到。 
                    从上例最后例子看出，m参数应该是和锚同时使用才有意思，否则直接去匹配也能匹配出来。说明m是在特殊需求下才使用的
               **/
               
              /*******************************参数s*********************************/
              //匹配value中包含m且之后为任意字符包括换行符并且还包含line字符的字符串
              temp.find( { description: { $regex: /m.*line/, $options: 'si' } } ) 
              /**
              结果为：
                  { "_id" : 102, "sku" : "xyz456", "description" : "Many spaces before     line" }
                  { "_id" : 103, "sku" : "xyz789", "description" : "Multiple\nline description" }
              **/
            }
        });

```
## 文档查询
使用Mongoose来查找文档很容易，有以下3种方法可供选择

- find()
- findById()
- findOne()
### find()
第一个参数表示查询条件，第二个参数用于控制返回的字段，第三个参数用于配置查询参数，第四个参数是回调函数，回调函数的形式为function(err,docs){}

```js
        Model.find(conditions, [projection], [options], [callback])
```
在数据库db1的集合temps中存在如下数据

<img  src="https://image-static.segmentfault.com/369/307/369307320-5a13bac055f67" alt="clipboard.png" title="clipboard.png" style="cursor: pointer; display: inline;">
现在，使用find()方法找出所有数据

```js
        var mongoose = require('mongoose');
        mongoose.connect("mongodb://u1:123456@localhost/db1", function(err) {
            if(!err){
                var schema = new mongoose.Schema({ age:Number, name: String});        
                var temp = mongoose.model('temp', schema);
                temp.find(function(err,docs){
                    //[ { _id: 5971f93be6f98ec60e3dc86c, name: 'huochai', age: 27 },
                    //{ _id: 5971f93be6f98ec60e3dc86d, name: 'wang', age: 18 },
                    //{ _id: 5971f93be6f98ec60e3dc86e, name: 'huo', age: 30 },
                    //{ _id: 5971f93be6f98ec60e3dc86f, name: 'li', age: 12 } ]
                    console.log(docs);
                })
            }
        });
```
找出年龄大于18的数据

```js
       temp.find({age:{$gte:18}},function(err,docs){
            //[ { _id: 5971f93be6f98ec60e3dc86c, name: 'huochai', age: 27 },
            //{ _id: 5971f93be6f98ec60e3dc86d, name: 'wang', age: 18 },
            //{ _id: 5971f93be6f98ec60e3dc86e, name: 'huo', age: 30 }]
            console.log(docs);
        })
```
找出年龄大于18且名字里存在'huo'的数据

```js
     temp.find({name:/huo/,age:{$gte:18}},function(err,docs){
                //[ { _id: 5971f93be6f98ec60e3dc86c, name: 'huochai', age: 27 },
                //{ _id: 5971f93be6f98ec60e3dc86e, name: 'huo', age: 30 }]
                console.log(docs);
            })
```
找出名字里存在'a'的数据，且只输出'name'字段

- [注意]_id字段默认输出

```js
         temp.find({name:/a/},'name',function(err,docs){
            //[ { _id: 5971f93be6f98ec60e3dc86c, name: 'huochai' },
            //{ _id: 5971f93be6f98ec60e3dc86d, name: 'wang' } ]
             console.log(docs);
          })
```
如果确实不需要_id字段输出，可以进行如下设置

 ```js
          temp.find({name:/a/},{name:1,_id:0},function(err,docs){
            //[ { name: 'huochai' }, { name: 'wang' } ]
             console.log(docs);
          })
```
找出跳过前两条数据的其他所有数据

- [注意]如果使用第三个参数，前两个参数如果没有值，需要设置为null

```js
        temp.find(null,null,{skip:2},function(err,docs){
            //[ { _id: 5971f93be6f98ec60e3dc86e, name: 'huo', age: 30 },
            //{ _id: 5971f93be6f98ec60e3dc86f, name: 'li', age: 12 } ]
            console.log(docs);
        })
```
### findById()

```js
        Model.findById(id, [projection], [options], [callback])
```
显示第0个元素的所有字段

```js
            var aIDArr = [];
            temp.find(function(err,docs){
                docs.forEach(function(item,index,arr){
                    aIDArr.push(item._id);
                })
                temp.findById(aIDArr[0],function(err,doc){
                    //{ _id: 5971f93be6f98ec60e3dc86c, name: 'huochai', age: 27 }
                    console.log(doc);
                })
            })
```
　　以上代码的另一种写法如下

```js
        var aIDArr = [];
        temp.find(function(err,docs){
            docs.forEach(function(item,index,arr){
                aIDArr.push(item._id);
            })
            temp.findById(aIDArr[0]).exec(function(err,doc){
                //{ _id: 5971f93be6f98ec60e3dc86c, name: 'huochai', age: 27 }
                console.log(doc);
            })
        })
```
只输出name字段

```js
        temp.findById(aIDArr[0],{name:1,_id:0},function(err,doc){
            //{  name: 'huochai'}
            console.log(doc);
        })
```
　　或者写成下面这种形式

```JS
        temp.findById(aIDArr[0],{name:1,_id:0}).exec(function(err,doc){
            //{  name: 'huochai'}
            console.log(doc);
        })
```
　　输出最少的字段

```js
        temp.findById(aIDArr[0],{lean:true},function(err,doc){
            //{ _id: 5971f93be6f98ec60e3dc86c }
            console.log(doc);
        })   
        temp.findById(aIDArr[0],{lean:true}).exec(function(err,doc){
            //{ _id: 5971f93be6f98ec60e3dc86c }
            console.log(doc);
        }) 
```
#### lean
Mongoose查询返回的对象是不能直接修改的

```js
        //  以下代码不能直接运行，仅给出流程
        const schema = new mongoose.Schema({
          title: String,
          date: Date,
          content: String
        });

        const Model = mongoose.model('test', schema);
        const docs =  await Model.find({});

        docs.forEach(function (doc) {
           doc.title = "测试";
        });

        // 执行完打印title，会发现修改不生效
        console.log(docs);

```
##### 修改的三种方法

```js
        // 不看文档直接撸
        const docs =  await Model.find({});
        const newDocs = docs.map(function (obj) {
          return Object.assign({}, obj);
        });

        // 调用toObject或toJSON方法
        const docs = (await Model.find({})).toObject();

        // 调用lean方法
        const docs = await Model.find({}).lean();
```
### findOne()
该方法返回查找到的所有实例的第一个

```js
        Model.findOne([conditions], [projection], [options], [callback])
```
　　找出age>20的文档中的第一个文档

```js
        temp.findOne({age:{$gt : 20}},function(err,doc){
            //{ _id: 5971f93be6f98ec60e3dc86c, name: 'huochai', age: 27 }
            console.log(doc);
        })   
        temp.findOne({age:{$gt : 20}}).exec(function(err,doc){
            //{ _id: 5971f93be6f98ec60e3dc86c, name: 'huochai', age: 27 }
            console.log(doc);
        }) 
```
　找出age>20的文档中的第一个文档，且只输出name字段

```js
        temp.findOne({age:{$gt : 20}},{name:1,_id:0},function(err,doc){
            //{ name: 'huochai' }
            console.log(doc);
        })   
        temp.findOne({age:{$gt : 20}},{name:1,_id:0}).exec(function(err,doc){
            //{ name: 'huochai' }
            console.log(doc);
        })
```
　　找出age>20的文档中的第一个文档，且输出包含name字段在内的最短字段

```js
        temp.findOne({age:{$gt : 20}},"name",{lean:true},function(err,doc){
            //{ _id: 5971f93be6f98ec60e3dc86c, name: 'huochai' }
            console.log(doc);
        })   
        temp.findOne({age:{$gt : 20}},"name").lean().exec(function(err,doc){
            //{ _id: 5971f93be6f98ec60e3dc86c, name: 'huochai' }
            console.log(doc);
        }) 
```
### $where
　　如果要进行更复杂的查询，需要使用$where操作符，$where操作符功能强大而且灵活，它可以使用任意的JavaScript作为查询的一部分，包含JavaScript表达式的字符串或者JavaScript函数

<img  src="https://image-static.segmentfault.com/314/702/3147025414-5a13bd7669d77"  style="cursor: pointer; display: inline;">
#### 使用字符串

```js
        temp.find({$where:"this.x == this.y"},function(err,docs){

        //[ { _id: 5972ed35e6f98ec60e3dc887,name: 'wang',age: 18,x: 1,y: 1 },
        //{ _id: 5972ed35e6f98ec60e3dc889, name: 'li', age: 20, x: 2, y: 2 } ]
        console.log(docs);
        }) 
        temp.find({$where:"obj.x == obj.y"},function(err,docs){

        //[ { _id: 5972ed35e6f98ec60e3dc887,name: 'wang',age: 18,x: 1,y: 1 },
        //{ _id: 5972ed35e6f98ec60e3dc889, name: 'li', age: 20, x: 2, y: 2 } ]
        console.log(docs);
        })
```

#### 使用函数

```js
        temp.find({$where:function(){
                return obj.x !== obj.y;
            }},function(err,docs){
            //[ { _id: 5972ed35e6f98ec60e3dc886,name: 'huochai',age: 27,x: 1,y: 2 },
            //{ _id: 5972ed35e6f98ec60e3dc888, name: 'huo', age: 30, x: 2, y: 1 } ]
            console.log(docs);
        }) 
        temp.find({$where:function(){
                return this.x !== this.y;
            }},function(err,docs){
            //[ { _id: 5972ed35e6f98ec60e3dc886,name: 'huochai',age: 27,x: 1,y: 2 },
            //{ _id: 5972ed35e6f98ec60e3dc888, name: 'huo', age: 30, x: 2, y: 1 } ]
            console.log(docs);
        })
```
## 文档更新
文档更新可以使用以下几种方法

- update()
- updateMany()
- find() + save()
- updateOne()
- findOne() + save()
- findByIdAndUpdate()
- fingOneAndUpdate()
### update()

```js
        Model.update(conditions, doc, [options], [callback])
```
- 第一个参数conditions为查询条件，
- 第二个参数doc为需要修改的数据，
- 第三个参数options为控制选项，
options有如下选项
    参数 | 说明
    --- | ---
   safe (boolean) |  默认为true。安全模式。
   upsert (boolean) |  默认为false。如果不存在则创建新记录。
   multi (boolean) |  默认为false。是否更新多个查询记录。
   runValidators |  如果值为true，执行Validation验证。
   setDefaultsOnInsert |  如果upsert选项为true，在新建时插入文档定义的默认值。
   strict (boolean) |  以strict模式进行更新。
   overwrite (boolean) |  默认为false。禁用update-only模式，允许覆盖记录。
- 第四个参数是回调函数

数据库temps中现有数据如下

<img src="https://image-static.segmentfault.com/807/809/807809902-5a13c2d012ec4" style="cursor: pointer; display: inline;">

现在使用update()方法查询age大于20的数据，并将其年龄更改为40岁

```js
        var mongoose = require('mongoose');
        mongoose.connect("mongodb://u1:123456@localhost/db1", function(err) {
            if(!err){
                var schema = new mongoose.Schema({ age:Number, name: String});
                var temp = mongoose.model('temp', schema);   
                temp.update({age:{$gte:20}},{age:40},function(err,raw){
                    //{ n: 1, nModified: 1, ok: 1 }
                    console.log(raw);
                })

            }
        });
```
经过以上操作，数据库结果如下。只有第一个数据更改为40岁。而第三个数据没有发生变化

<img src="https://image-static.segmentfault.com/223/009/2230093202-5a13c30c3cd1d" style="cursor: pointer; display: inline;">

如果要同时更新多个记录，需要设置options里的multi为true。下面将名字中有'a'字符的年龄设置为10岁

```js
        var mongoose = require('mongoose');
        mongoose.connect("mongodb://u1:123456@localhost/db1", function(err) {
            if(!err){
                var schema = new mongoose.Schema({ age:Number, name: String});        
                var temp = mongoose.model('temp', schema);   
                temp.update({name:/a/},{age: 10},{multi:true},function(err,raw){
                    //{ n: 2, nModified: 2, ok: 1 }
                    console.log(raw);
                })

            }           
        });
```

<img src="https://image-static.segmentfault.com/308/843/3088436051-5a13c330d9ca1" style="cursor: pointer; display: inline;">

如果设置的查找条件，数据库里的数据并不满足，默认什么事都不发生

```js
        temp.update({age:100},{name: "hundred"},function(err,raw){
            //{ n: 0, nModified: 0, ok: 1 }
            console.log(raw);
        })
```
如果设置options里的upsert参数为true，若没有符合查询条件的文档，mongo将会综合第一第二个参数向集合插入一个新的文档

```js
        temp.update({age:100},{name: "hundred"},{upsert:true},function(err,raw){
            //{ n: 1, nModified: 0,upserted: [ { index: 0, _id: 5972c202d46b621fca7fc8c7 } ], ok: 1 }
            console.log(raw);
        })
```
<img src="https://image-static.segmentfault.com/105/684/1056845692-5a13c3675f64e" style="cursor: pointer; display: inline;">

```js
        temp.update({name:/aa/},{age: 0},{upsert:true},function(err,raw){
            //{ n: 1, nModified: 0,upserted: [ { index: 0, _id: 5972c288d46b621fca7fdd8f } ], ok: 1 }
            console.log(raw);
        })
```

<img src="https://image-static.segmentfault.com/219/806/2198065275-5a13c3889b3b5" style="cursor: pointer; display: inline;">

- [注意]update()方法中的回调函数不能省略，否则数据不会被更新。如果回调函数里并没有什么有用的信息，则可以使用exec()简化代码

```js
        temp.update({name:/aa/},{age: 0},{upsert:true}).exec();
```

### updateMany()
- updateMany()与update()方法唯一的区别就是默认更新多个文档，即使设置{multi:false}也无法只更新第一个文档

```js
        Model.updateMany(conditions, doc, [options], [callback])
```
将数据库中名字中带有'huo'的数据，年龄变为50岁

```js
        temp.updateMany({name:/huo/},{age:50},function(err,raw){
            //{ n: 2, nModified: 2, ok: 1 }
            console.log(raw);
        });
```
<img src="https://image-static.segmentfault.com/393/948/3939483079-5a13c3d936fc8" style="cursor: pointer; display: inline;">

### find() + save()
如果需要更新的操作比较复杂，可以使用find()+save()方法来处理，比如找到年龄小于30岁的数据，名字后面添加'30'字符

```js
        temp.find({age:{$lt:30}},function(err,docs){
            //[ { _id: 5971f93be6f98ec60e3dc86d, name: 'wang', age: 10 },
            //{ _id: 5971f93be6f98ec60e3dc86f, name: 'li', age: 12 }]
            console.log(docs);
            docs.forEach(function(item,index,arr){
                item.name += '30';
                item.save();
            })
            //[ { _id: 5971f93be6f98ec60e3dc86d, name: 'wang30', age: 10 },
            // { _id: 5971f93be6f98ec60e3dc86f, name: 'li30', age: 12 }]
            console.log(docs);
        });
```
### updateOne()
- updateOne()方法只能更新找到的第一条数据，即使设置{multi:true}也无法同时更新多个文档

将数据库中名字中带有'huo'的数据，年龄变为60岁

```js
        temp.updateOne({name:/huo/},{age:60},function(err,raw){
            //{ n: 1, nModified: 1, ok: 1 }
            console.log(raw);
        });
```

<img src="https://image-static.segmentfault.com/393/948/3939483079-5a13c3d936fc8" style="cursor: pointer; display: inline;">

### findOne() + save()
　　如果需要更新的操作比较复杂，可以使用findOne()+save()方法来处理，比如找到名字为'huochai'的数据，年龄加100岁

```js
        temp.findOne({name:'huochai'},function(err,doc){
            //{ _id: 5971f93be6f98ec60e3dc86c, name: 'huochai', age: 10 }
            console.log(doc);
            doc.age += 100;
            doc.save();
            //{ _id: 5971f93be6f98ec60e3dc86c, name: 'huochai', age: 110 }
            console.log(doc);
        });
```
findOneAndUpdate()
　　fineOneAndUpdate()方法的第四个参数回调函数的形式如下function(err,doc){}

```js
        Model.findOneAndUpdate([conditions], [update], [options], [callback])
```
findByIdAndUpdate
　　 fineByIdAndUpdate()方法的第四个参数回调函数的形式如下function(err,doc){}

```js
        Model.findOneAndUpdate([conditions], [update], [options], [callback])
```

## 文档删除
有三种方法用于文档删除

- remove()
- findOneAndRemove()
- findByIdAndRemove()
### remove()
remove有两种形式，一种是文档的remove()方法，一种是Model的remove()方法

- 下面介绍Model的remove()方法，该方法的第一个参数conditions为查询条件，第二个参数回调函数的形式如下function(err){}　　

```js
        model.remove(conditions, [callback])
```
<img src="https://image-static.segmentfault.com/266/683/2666832813-5a13c5e6e4182" style="cursor: pointer; display: inline;">

　　删除数据库中名称包括'30'的数据

```js
        temp.remove({name:/30/},function(err){})
        //[注意]remove()方法中的回调函数不能省略，否则数据不会被删除。当然，可以使用exec()方法来简写代码
        temp.remove({name:/30/}).exec()
```
结果为:

<img src="https://image-static.segmentfault.com/287/807/287807368-5a13c60c7102a" style="cursor: pointer; display: inline;">

- 下面介绍文档的remove()方法，该方法的参数回调函数的形式如下function(err,doc){}

```JS
        document.remove([callback])
```
　　删除数据库中名称包含'huo'的数据

　　[注意]文档的remove()方法的回调函数参数可以省略

```JS
        temp.find({name:/huo/},function(err,doc){
            doc.forEach(function(item,index,arr){
                item.remove(function(err,doc){
                    //{ _id: 5971f93be6f98ec60e3dc86c, name: 'huochai', age: 30 }
                    //{ _id: 5971f93be6f98ec60e3dc86e, name: 'huo', age: 60 }
                    console.log(doc);
                })
            })
        }) 
```

### findOneAndRemove()
model的remove()会删除符合条件的所有数据，如果只删除符合条件的第一条数据，则可以使用model的findOneAndRemove()方法

```js
        Model.findOneAndRemove(conditions, [options], [callback])
```
　　集合temps现有数据如下

<img src="https://image-static.segmentfault.com/401/912/4019122241-5a13c6dfc0556" style="cursor: pointer; display: inline;">

　现在删除第一个年龄小于20的数据

```js
        temp.findOneAndRemove({age:{$lt:20}},function(err,doc){
            //{ _id: 5972d3f3e6f98ec60e3dc873, name: 'wang', age: 18 }
            console.log(doc);
        })
        //与model的remove()方法相同，回调函数不能省略，否则数据不会被删除。当然，可以使用exec()方法来简写代码
        temp.findOneAndRemove({age:{$lt:20}}).exec()
```
结果为:

<img src="https://image-static.segmentfault.com/506/503/506503687-5a13c7000be76" style="cursor: pointer; display: inline;">


### findByIdAndRemove()

```JS
    Model.findByIdAndRemove(id, [options], [callback])
```

<img src="https://image-static.segmentfault.com/185/629/1856298344-5a13c74639000" style="cursor: pointer; display: inline;">

删除第0个元素

```js
        var aIDArr = [];
        temp.find(function(err,docs){
            docs.forEach(function(item,index,arr){
                aIDArr.push(item._id);
            })
            temp.findByIdAndRemove(aIDArr[0],function(err,doc){
                //{ _id: 5972d754e6f98ec60e3dc882, name: 'huochai', age: 27 }
                console.log(doc);
            })
        })
```

<img src="https://image-static.segmentfault.com/254/421/2544218912-5a13c761cf49e" style="cursor: pointer; display: inline;">

　类似的，该方法也不能省略回调函数，否则数据不会被删除。当然，可以使用exec()方法来简写代码

```js
        var aIDArr = [];
        temp.find(function(err,docs){
            docs.forEach(function(item,index,arr){
                aIDArr.push(item._id);
            })
            temp.findByIdAndRemove(aIDArr[0]).exec()
        })
```
## 前后钩子
　前后钩子即pre()和post()方法，又称为中间件，是在执行某些操作时可以执行的函数。中间件在schema上指定，类似于静态方法或实例方法等

　　可以在数据库执行下列操作时，设置前后钩子

-  validate
-  save
-  remove
-  count
-  find
-  findOne
-  findOneAndRemove
-  findOneAndUpdate
-  insertMany
-  update
### pre()
　　以find()方法为例，在执行find()方法之前，执行pre()方法

```js
        var schema = new mongoose.Schema({ age:Number, name: String,x:Number,y:Number});  
        schema.pre('find',function(next){
            console.log('我是pre方法1');
            next();
        });
        schema.pre('find',function(next){
            console.log('我是pre方法2');
            next();
        });  
        var temp = mongoose.model('temp', schema);
        temp.find(function(err,docs){
            console.log(docs[0]);
        })    
        /*
        我是pre方法1
        我是pre方法2
        { _id: 5972ed35e6f98ec60e3dc886,name: 'huochai',age: 27,x: 1,y: 2 }
        */
```
### post()
　　post()方法并不是在执行某些操作后再去执行的方法，而在执行某些操作前最后执行的方法，post()方法里不可以使用next()

```js
        var schema = new mongoose.Schema({ age:Number, name: String,x:Number,y:Number});  
        schema.post('find',function(docs){
            console.log('我是post方法1');
        });
        schema.post('find',function(docs){
            console.log('我是post方法2');
        });
        var temp = mongoose.model('temp', schema);
        temp.find(function(err,docs){
            console.log(docs[0]);
        }) 
        /*
        我是post方法1
        我是post方法2
        { _id: 5972ed35e6f98ec60e3dc886,name: 'huochai',age: 27,x: 1,y: 2 }
         */  
```
## 查询后处理
常用的查询后处理的方法如下所示
方法 | 作用
--- | ---
sort | 排序
skip | 跳过
limit | 限制
select | 显示字段
exect | 执行
count | 计数
distinct | 去重

```js
        var schema = new mongoose.Schema({ age:Number, name: String,x:Number,y:Number});  
        var temp = mongoose.model('temp', schema);
        temp.find(function(err,docs){
            //[ { _id: 5972ed35e6f98ec60e3dc886,name: 'huochai',age: 27,x: 1,y: 2 },
            //{ _id: 5972ed35e6f98ec60e3dc887,name: 'wang',age: 18,x: 1,y: 1 },
            //{ _id: 5972ed35e6f98ec60e3dc888, name: 'huo', age: 30, x: 2, y: 1 },
            //{ _id: 5972ed35e6f98ec60e3dc889, name: 'li', age: 20, x: 2, y: 2 } ]
            console.log(docs);
        }) 
```
### sort()
　　按age从小到大排序

```js
        temp.find().sort("age").exec(function(err,docs){
            //[ { _id: 5972ed35e6f98ec60e3dc887,name: 'wang',age: 18,x: 1,y: 1 },
            //{ _id: 5972ed35e6f98ec60e3dc889, name: 'li', age: 20, x: 2, y: 2 },
            //{ _id: 5972ed35e6f98ec60e3dc886,name: 'huochai',age: 27,x: 1,y: 2 },
            //{ _id: 5972ed35e6f98ec60e3dc888, name: 'huo', age: 30, x: 2, y: 1 } ]
            console.log(docs);
        }); 
```
　　按x从小到大，age从大到小排列

```js
        //{"X":1,"age":-1} 1升序 -1 降序
        temp.find().sort("x -age").exec(function(err,docs){
            //[ { _id: 5972ed35e6f98ec60e3dc886,name: 'huochai',age: 27,x: 1,y: 2 },
            //{  _id: 5972ed35e6f98ec60e3dc887,name: 'wang',age: 18,x: 1,y: 1 },
            //{ _id: 5972ed35e6f98ec60e3dc888, name: 'huo', age: 30, x: 2, y: 1 },
            //{ _id: 5972ed35e6f98ec60e3dc889, name: 'li', age: 20, x: 2, y: 2 } ]
            console.log(docs);
        });
```
### skip()
　　跳过1个，显示其他

```js
        temp.find().skip(1).exec(function(err,docs){
            //[ { _id: 5972ed35e6f98ec60e3dc887,name: 'wang',age: 18,x: 1,y: 1 },
            //{ _id: 5972ed35e6f98ec60e3dc888, name: 'huo', age: 30, x: 2, y: 1 },
            //{ _id: 5972ed35e6f98ec60e3dc889, name: 'li', age: 20, x: 2, y: 2 } ]
            console.log(docs);
        });
```

### limit()
　　显示2个

```js
        temp.find().limit(2).exec(function(err,docs){
            //[ { _id: 5972ed35e6f98ec60e3dc886,name: 'huochai',age: 27,x: 1,y: 2 },
            //{ _id: 5972ed35e6f98ec60e3dc887,name: 'wang',age: 18,x: 1,y: 1 } ]
            console.log(docs);
        }); 
```
### select()
　　显示name、age字段，不显示_id字段

```js
        temp.find().select("name age -_id").exec(function(err,docs){
            //[ { name: 'huochai', age: 27 },{ name: 'wang', age: 18 },{ name: 'huo', age: 30 },{ name: 'li', age: 20 } ]
            console.log(docs);
        }); 
        temp.find().select({name:1, age:1, _id:0}).exec(function(err,docs){
            //[ { name: 'huochai', age: 27 },{ name: 'wang', age: 18 },{ name: 'huo', age: 30 },{ name: 'li', age: 20 } ]
            console.log(docs);
        }); 
```
　　下面将以上方法结合起来使用，跳过第1个后，只显示2个数据，按照age由大到小排序，且不显示_id字段

```js
        temp.find().skip(1).limit(2).sort("-age").select("-_id").exec(function(err,docs){
            //[ { name: 'huochai', age: 27, x: 1, y: 2 },
            //{ name: 'li', age: 20, x: 2, y: 2 } ]
            console.log(docs);
        }); 
```
### count()
　　显示集合temps中的文档数量

```js
        temp.find().count(function(err,count){
            console.log(count);//4
        });
```
### distinct()
　　返回集合temps中的x的值

```js
        temp.find().distinct('x',function(err,distinct){
            console.log(distinct);//[ 1, 2 ]
        }); 
```
## 文档验证
为什么需要文档验证呢？以一个例子作为说明，schema进行如下定义

```js
        var schema = new mongoose.Schema({ age:Number, name: String,x:Number,y:Number});
```
　　如果不进行文档验证，保存文档时，就可以不按照Schema设置的字段进行设置，分为以下几种情况

　　1、缺少字段的文档可以保存成功

```js
        var temp = mongoose.model('temp', schema);
        new temp({age:10}).save(function(err,doc){
            //{ __v: 0, age: 10, _id: 597304442b70086a1ce3cf05 }
            console.log(doc);
        }); 
```
　　2、包含未设置的字段的文档也可以保存成功，未设置的字段不被保存

```js
        new temp({age:100,abc:"abc"}).save(function(err,doc){
            //{ __v: 0, age: 100, _id: 5973046a2bb57565b474f48b }
            console.log(doc);
        });
```
　　3、包含字段类型与设置不同的字段的文档也可以保存成功，不同字段类型的字段被保存为设置的字段类型

```js
        new temp({age:true,name:10}).save(function(err,doc){
            //{ __v: 0, age: 1, name: '10', _id: 597304f7a926033060255366 }
            console.log(doc);
        }); 
```
　　而通过文档验证，就可以避免以下几种情况发生

　　文档验证在SchemaType中定义，格式如下

```js
        {name: {type:String, validator:value}}
```
　　常用验证包括以下几种

- required: 数据必须填写
- default: 默认值
- validate: 自定义匹配
- min: 最小值(只适用于数字)
- max: 最大值(只适用于数字)
- match: 正则匹配(只适用于字符串)
- enum:  枚举匹配(只适用于字符串)
### required
　　将age设置为必填字段，如果没有age字段，文档将不被保存，且出现错误提示

```js
        var schema = new mongoose.Schema({ age:{type:Number,required:true}, name: String,x:Number,y:Number});  
        var temp = mongoose.model('temp', schema);
        new temp({name:"abc"}).save(function(err,doc){
            //Path `age` is required.
            console.log(err.errors['age'].message);
        }); 
```
### default
　　设置age字段的默认值为18，如果不设置age字段，则会取默认值

```js
        var schema = new mongoose.Schema({ age:{type:Number,default:18}, name:String,x:Number,y:Number});  
        var temp = mongoose.model('temp', schema);
        new temp({name:'a'}).save(function(err,doc){
            //{ __v: 0, name: 'a', _id: 59730d2e7a751d81582210c1, age: 18 }
            console.log(doc);
        }); 
```
### min | max
　　将age的取值范围设置为[0,10]。如果age取值为20，文档将不被保存，且出现错误提示

```js
        var schema = new mongoose.Schema({ age:{type:Number,min:0,max:10}, name: String,x:Number,y:Number});  
        var temp = mongoose.model('temp', schema);
        new temp({age:20}).save(function(err,doc){
            //Path `age` (20) is more than maximum allowed value (10).
            console.log(err.errors['age'].message);
        });
```
### match
　　将name的match设置为必须存在'a'字符。如果name不存在'a'，文档将不被保存，且出现错误提示

```js
        var schema = new mongoose.Schema({ age:Number, name:{type:String,match:/a/},x:Number,y:Number});  
        var temp = mongoose.model('temp', schema);
        new temp({name:'bbb'}).save(function(err,doc){
            //Path `name` is invalid (bbb).
            console.log(err.errors['name'].message);
        }); 
```
### enum
　　将name的枚举取值设置为['a','b','c']，如果name不在枚举范围内取值，文档将不被保存，且出现错误提示

```js
        var schema = new mongoose.Schema({ age:Number, name:{type:String,enum:['a','b','c']},x:Number,y:Number});  
        var temp = mongoose.model('temp', schema);
        new temp({name:'bbb'}).save(function(err,doc){
            //`bbb` is not a valid enum value for path `name`.
            console.log(err.errors['name'].message);

        });
```
### validate
　　validate实际上是一个函数，函数的参数代表当前字段，返回true表示通过验证，返回false表示未通过验证。利用validate可以自定义任何条件。比如，定义名字name的长度必须在4个字符以上

```js
        var validateLength = function(arg){
            if(arg.length > 4){
                return true;
            }
            return false;
        };
        var schema = new mongoose.Schema({ name:{type:String,validate:validateLength}, age:Number,x:Number,y:Number});  
        var temp = mongoose.model('temp', schema);
        new temp({name:'abc'}).save(function(err,doc){
            //Validator failed for path `name` with value `abc`
            console.log(err.errors['name'].message);
        });
```
# 练习
本节引用 mongoose的基本使用 有删改

## 连接数据库
编辑 test.js ：

```js
        var mongoose = require('mongoose');
        var db = mongoose.connect('mongodb://127.0.0.1:27017/test');
        db.connection.on('error', function(error){
          console.log('数据库test连接失败：' + error);
        });
        db.connection.on('open', function(){
          console.log('数据库test连接成功');
        });
```
接着先打开一个 iTerm2 终端，开启 mongodb 服务：

```js
        mongod
```

再打开另一个 iTerm2 终端，运行 test.js：

```js
        node test.js
        //成功后便会输出：数据库test连接成功
```
## Schema/Model/Entity
没有比文档更详细的了：http://mongoosejs.com/docs/gu...

- Schema：数据库集合的结构对象。
- Model ：由Schema构造而成，可操作数据库。
- Entity：由Model创建的实体，可操作数据库。

看完文档后，再看看下面一段代码配合理解一下：

```js
        var mongoose = require("mongoose");
        var db = mongoose.connect("mongodb://127.0.0.1:27017/test");
        // var testModel = db.model('test1', testSchema); // 集合名称；集合的结构对象
        var TestSchema = new mongoose.Schema({
            name : { type:String },
            age  : { type:Number, default:0 },
            email: { type:String },
            time : { type:Date, default:Date.now }
        });
        var TestModel = db.model("test1", TestSchema );
        var TestEntity = new TestModel({
            name : "helloworld",
            age  : 28,
            email: "helloworld@qq.com"
        });
        TestEntity.save(function(error,doc){
          if(error){
             console.log("error :" + error);
          }else{
             console.log(doc);
          }
        });
```
## model 数据插入
在前面的数据库连接成功的前提下，我们在数据库 test 下新建一个集合 test1 、并往里面插入保存一组数据：

```js
        var testSchema = new mongoose.Schema({
          name: {type: String},
          age: {type: Number, default: 0},
          email: {type: String},
          time: {type: Date, default: Date.now}
        });
        var testModel = db.model('test1', testSchema); // 集合名称；集合的结构对象
        // Document文档（关联数组式的对象） < Collection集合 < 数据库
        // 插入保存一段数据
        testModel.create([
          {name: "test1", age: 8},
          {name: "test2", age: 18},
          {name: "test3", age: 28},
          {name: "test4", age: 38},
          {name: "test5", age: 48},
          {name: "test6", age: 58, email:"tttt@qq.com"},
          {name: "test7", age: 68, email:"ssss@qq.com"},
          {name: "test8", age: 18},
          {name: "test9", age: 18, email:"rrrr@qq.com"},
          {name: "test10",age: 18}
        ], function (error, docs) {
          if(error) {
            console.log(error);
          } else {
            console.log('save ok');
            console.log(docs);
          }
        });
```
## find 数据查询
mongoose 提供了find、findOne、和findById方法用于文档查询。
基本语法：

```js
        model.find(Conditions,fields,options,callback(err, doc));
        // Conditions: 查询条件
        // fields: 返回的字段
        // options: 游标（sort,limit）
        // callback: 回调函数，参数doc为查询出来的结果
```
条件查询的基础：

```js
        $lt (小于<)
        $lte (小于等于<=)
        $gt (大于>)
        $gte (大于等于>=)
        $ne (不等于,不包含!=)
        $in (包含)
        $or (查询多个键值的任意给定值)
        $exists (判断某些属性是否存在)
        $all (全部)
```

具体的一些实例，代码里已有详细注释：

```js
        // find(Conditions,fields,callback);
        // 省略或为空、返回所有记录；只包含name,age字段，去掉默认的_id字段；执行回调函数
        testModel.find({}, {name:1, age:1, _id:0}, function(err, docs){
          if (err) {
            console.log('查询出错：' + err);
          } else {
            console.log('{}查询结果为：');
            console.log(docs);
          }
        });
        ----
        {}查询结果为：
        [ { name: 'test3', age: 28 },
          { name: 'test2', age: 18 },
          { name: 'test1', age: 8 },
          { name: 'test6', age: 58 },
          { name: 'test4', age: 38 },
          { name: 'test7', age: 68 },
          { name: 'test8', age: 18 },
          { name: 'test9', age: 18 },
          { name: 'test5', age: 48 },
          { name: 'test10', age: 18 } ]
        ----
        // 查询age大于等于28，小于等于48
        testModel.find({age: {$gte: 28, $lte: 48}}, {name:1, age:1, _id:0}, function(err, docs){
          if (err) {
            console.log('查询出错：' + err);
          } else {
            console.log('$gte,$lte查询结果为：');
            console.log(docs);
          }
        });
        ----
        $gte,$lte查询结果为：
        [ { name: 'test3', age: 28 },
          { name: 'test4', age: 38 },
          { name: 'test5', age: 48 } ]
        ----
        // 查询age为58、68的2条数据
        testModel.find({age: {$in: [58, 68]}}, {name:1, age:1, _id:0}, function(err, docs){
          if (err) {
            console.log('查询出错：' + err);
          } else {
            console.log('$in查询结果为：');
            console.log(docs);
          }
        });
        ----
        $in查询结果为：
        [ { name: 'test6', age: 58 }, { name: 'test7', age: 68 } ]
        ----
        // 查询name为test3、或者age为18的全部数据
        testModel.find({$or: [{name: 'test3'}, {age: 18}]}, {name:1, age:1, _id:0}, function(err, docs){
          if (err) {
            console.log('查询出错：' + err);
          } else {
            console.log('$or查询结果为：');
            console.log(docs);
          }
        });
        ----
        $or查询结果为：
        [ { name: 'test3', age: 28 },
          { name: 'test2', age: 18 },
          { name: 'test8', age: 18 },
          { name: 'test9', age: 18 },
          { name: 'test10', age: 18 } ]
        ----

        // step3：游标查询
        // 查询name为test3、或者age为18的全部数据；但限制只查询2条数据
        testModel.find({$or: [{name: 'test3'}, {age: 18}]}, {name:1, age:1, _id:0}, {limit: 2}, function(err, docs){
          if (err) {
            console.log('查询出错：' + err);
          } else {
            console.log('limit查询结果为：');
            console.log(docs);
          }
        });
        ----
        limit查询结果为：
        [ { name: 'test3', age: 28 }, { name: 'test2', age: 18 } ]
        ----

        //查询age大于等于28，小于等于48;降序输出
        testModel.find({age: {$gte: 28, $lte: 48}}, {name:1, age:1, _id:0}, {sort: {age: -1}}, function(err, docs){
          if (err) {
            console.log('查询出错：' + err);
          } else {
            console.log('sort查询结果为：');
            console.log(docs);
          }
        });
        ----
        sort查询结果为：
        [ { name: 'test5', age: 48 },
          { name: 'test4', age: 38 },
          { name: 'test3', age: 28 } ]
        ----
```
## update 数据更新
基本使用：model.update(查询条件,更新对象,callback);

```js
        var conditions = {name: 'test1'};
        var update = {$set: {age: 11 }};
        testModel.update(conditions, update, function(error){
          if(error) {
            console.log(error);
          } else {
            console.log('Update success!');
            testModel.find({name: 'test1'}, {name:1, age:1, _id:0}, function(err, docs){
              if (err) {
                console.log('查询出错：' + err);
              } else {
                console.log('更新test1后的查询结果为：');
                console.log(docs);  
                // 更新test_update后的查询结果为空数组：[ ];
                // 更新test1后的查询结果为: [ { name: 'test1', age: 11 } ]
                // 只能更新本来已存在的数据
              }
            });
          }
        ---
        Update success!
        更新test1后的查询结果为：
        [ { name: 'test1', age: 11 } ]
        ---
```
## remove 数据删除
基本使用：model.remove(查询条件,callback);

```js
        var conditions = {name: 'test2'};
        testModel.remove(conditions, function(error){
          if(error) {
            console.log(error);
          } else {
            console.log('Delete success!');
            testModel.find({name: 'test2'}, {name:1, age:1, _id:0}, function(err, docs){
              if (err) {
                console.log('查询出错：' + err);
              } else {
                console.log('删除test2后的查询结果为：');
                console.log(docs);  // 删除test2后的查询结果为空数组：[ ];
              }
            });
          }
        });
        ----
        Delete success!
        删除test2后的查询结果为：
        []
        ----
```
