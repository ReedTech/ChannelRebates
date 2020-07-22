[abp](https://abp.io) vnext v2.9.0 [Blogging模块](https://github.com/abpframework/abp/tree/2.9.0/modules/blogging)安装使用过程中，遇到TuiEditor脚本报错和添加post上传封面报错2个问题。

## tuiEditor报错

**2020-07-22更新：非abp的问题造成，应该是我还原客户端js包的过程中出了问题，在[梁士伟](https://liangsw.me/) 同学帮助下，发现是tui-editor依赖的tui-code-snippet包的版本不一致造成。**

> tuiEditor v1.4.1之后默认不包括jquery扩展，abp v2.9引用版本为v1.4.10，不能使用jquery扩展方法初始化。

> 另外，abp引用js为tui-editor-Editor.min.js无法创建编辑器，通过重写TuiEditorScriptContributor替换js为tui-editor-Editor-full.min.js

```` csharp
public class TuiEditorExtensionScriptContributor : BundleContributor
    {
        public override void ConfigureBundle(BundleConfigurationContext context)
        {
            context.Files.ReplaceOne(
                "/libs/tui-editor/tui-editor-Editor.min.js",
                "/libs/tui-editor/tui-editor-Editor-full.min.js"
            );
        }
    }
````

在web模块的`ConfuguireServices`中添加如下代码：

```` csharp
Configure<AbpBundlingOptions>(options =>
{
    options.ScriptBundles
        .Configure(
            typeof(Volo.Blogging.Pages.Blog.Posts.NewModel).FullName,
            bundleConfig =>
            {
                bundleConfig.AddContributors(typeof(TuiEditorExtensionScriptContributor));
            })
        .Configure(
            typeof(Volo.Blogging.Pages.Blog.Posts.EditModel).FullName,
            bundleConfig =>
            {
                bundleConfig.AddContributors(typeof(TuiEditorExtensionScriptContributor));
            });
});

````

> 并重写Pages/Blogs/Posts/new.js和Pages/Blogs/Posts/edit.js编辑器初始化代码如下：

```` javascript
var newPostEditor = new tui.Editor({
        el: $editorContainer.get(0),
        usageStatistics: false,
        initialEditType: 'markdown',
        previewStyle: 'tab',
        height: "auto",
        hooks: {
            addImageBlobHook: function (blob, callback, source) {
                var imageAltText = blob.name;

                uploadImage(blob,
                    function (fileUrl) {
                        callback(fileUrl, imageAltText);
                    });
            }
        },
        events: {
            load: function () {
                $editorContainer.find(".loading-cover").remove();
                $submitButton.prop("disabled", false);
                $form.data("validator").settings.ignore = '.ignore';
                $editorContainer.find(':input').addClass('ignore');
            }
        }
    })

    $editorContainer.data(editorDataKey);
````

## 上传封面图报错问题

> 报错是因为没有配置上传文件的保存路径

在web模块的`ConfuguireServices`中调用如下方法：

```` csharp
/// <summary>
/// 配置Blog文件存储路径
/// </summary>
/// <param name="hostingEnvironment"></param>
private void ConfigureBlogFileOptions(IWebHostEnvironment hostingEnvironment)
{
     Configure<BlogFileOptions>(options =>
     {
          options.FileUploadLocalFolder = Path.Combine(hostingEnvironment.WebRootPath, "files");
      });
}
````