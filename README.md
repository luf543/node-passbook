# Get your certificates
获取证书

To start with, you'll need a certificate issued by [the iOS Provisioning
Portal](https://developer.apple.com/ios/manage/passtypeids/index.action).  You
need one certificate per Passbook Type ID.
首先，您需要一个由iOS配置门户颁发的证书。你每Passbook Type ID需要一个证书.

After adding this certificate to your Keychain, you need to export it as a
`.p12` file and copy it into the keys directory.
添加此证书到您的钥匙串后，你需要将它导出为'.P12'文件并将其复制到密钥目录。

You will also need the 'Apple Worldwide Developer Relations Certification
Authority' certificate and to conver the `.p12` files into `.pem` files.  You
can do both using the `node-passbook prepare-keys` command:
你也需要"苹果全球开发者关系认证机构"证书并将'.p12'文件转换成'.pem'文件。你可以使用'node-passbook prepare-keys'命令：

```sh
node-passbook prepare-keys -p keys
```

This is the same directory into which you placet the `.p12` files.
和你生成的'.p12'文件在同一个目录下


# Start with a template
从模板开始

Start with a template.  A template has all the common data fields that will be
shared between your passes, and also defines the keys to use for signing it.
从模板开始,模板具有所有的公共数据字段，这些数据字段将在您的passes之间共享，并且还定义了用于签名的密钥。

```js
var createTemplate = require("passbook");

var template = createTemplate("coupon", {
  passTypeIdentifier: "pass.com.example.passbook",
  teamIdentifier:     "MXL",
  backgroundColor:   "rgb(255,255,255)"
});
```

The first argument is the pass style (`coupon`, `eventTicket`, etc), and the
second optional argument has any fields you want to set on the template.
第一个参数是pass样式（优惠券，eventticket，等），第二可选参数是你想设置模板上的任意数据。

You can access template fields directly, or from chained accessor methods, e.g:
你可以直接赋值模板字段，或链式的访问方法，如：

```js
template.fields.passTypeIdentifier = "pass.com.example.passbook";

console.log(template.passTypeIdentifier());

template.teamIdentifier("MXL").
  passTypeIdentifier("pass.com.example.passbook")
```

The following template fields are required:
下面的模板字段是必需的：
`passTypeIdentifier`  - The Passbook Type ID, begins with "pass."
`teamIdentifier`      - May contain an I

Optional fields that you can set on the template (or pass): 
可以在模板上设置的可选字段（或通过）：
`backgroundColor`,
`foregroundColor`, `labelColor`, `logoText`, `organizationName`,
`suppressStripShine` and `webServiceURL`.

In addition, you need to tell the template where to find the key files and where
to load images from:
此外，您还需要告诉模板在哪里找到密钥文件，以及从何处加载图像：

```js
template.keys("/etc/passbook/keys", "secret");
template.loadImagesFrom("images");
```

The last part is optional, but if you have images that are common to all passes,
you may want to specify them once in the template.
最后一部分是可选的，但如果你有所有passes的图像，你可能想在模板中指定一次。


# Create your pass
创建你的pass

To create a new pass from a template:
从一个模版创建一个新的pass

```js
var pass = template.createPass({
  serialNumber:  "123456",
  description:   "20% off"
});
```

Just like template, you can access pass fields directly, or from chained
accessor methods, e.g:
就像模版一样，你可以通过直接赋值字段，或链式的访问方法，如：

```js
pass.fields.serialNumber = "12345";
console.log(pass.serialNumber());
pass.serialNumber("12345").
  description("20% off");
```

In the JSON specification, structure fields (primary fields, secondary fields,
etc) are represented as arrays, but items must have distinct key properties.  Le
sigh.
在JSON规范中，结构字段（主字段、次要字段等）表示为数组，但项必须具有不同的关键属性。叹气。

To make it easier, you can use methods like `add`, `get` and `remove` that
will do the logical thing.  For example, to add a primary field:
为了使它更容易，您可以使用方法，如添加，获取和删除，将做逻辑的事情。例如，添加一个主字段：

```js
pass.primaryFields.add("date", "Date", "Nov 1");
pass.primaryFields.add({ key: "time", label: "Time", value: "10:00AM");
```

You can also call `add` with an array of triplets or array of objects.
您也可以调用添加一个三维数组或数组对象。

To get one or all fields:
得到一个或全部字段：

```js
var dateField = pass.primaryFields.get("date");
var allFields = pass.primaryFields.all();
```

To remove one or all fields:
删除一个或全部字段：

```js
pass.primaryFields.remove("date");
pass.primaryFields.clear();
```

Adding images to a pass is the same as adding images to a template:
添加图片到pass和添加图片到模版的一样:

```js
pass.images.icon = iconFilename;
pass.icon(iconFilename);
pass.loadImagesFrom("images");
```

You can add the image itself (a `Buffer`), or provide the name of a file or an
HTTP/S URL for retrieving the image.  You can also provide a function that will
be called when it's time to load the image, and should pass an error, or `null`
and a buffer to its callback.
您可以添加图像本身（缓冲区），或为检索图像提供文件名或http / s URL。
您还可以提供一个函数，该函数在加载图像时将被调用，并且应该传递一个错误，或NULL和它回调的缓冲区。


# Generate the file
生成的文件

To generate a file:
生成的文件:

```js
var file = fs.createWriteStream("mypass.pkpass");
pass.on("error", function(error) {
  console.error(error);
  process.exit(1);
})
pass.pipe(file);
```

Your pass will emit the `error` event if it fails to generate a valid Passbook
file, and emit the `end` event when it successfuly completed generating the
file.
你的pass如果不能生成有效的passbook文件会输出'error'事件，否则顺利完成生成文件输出'end'。

You can pipe to any writeable stream.  When working with HTTP, the `render`
method will set the content type, pipe to the HTTP response, and make use of a
callback (if supplied).
你可以交互任何可写流。当使用HTTP时，'render'方法将内容类型、交互设置为HTTP响应，并使用回调（如果提供）。

```js
server.get("/mypass", function(request, response) {
  pass.render(response, function(error) {
    if (error)
      console.error(error);
  });
});
```
