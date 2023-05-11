# React实现多入口SPA

> react一般用来做单页面应用，但是随着工程越来越大，一打包好几兆，严重影响加载速度。这时候就需要多入口打包，将工程分为几个小的SPA。比如登录时一个SPA，内容管理是一个SPA，产品管理是一个SPA，每个SPA加载共同的JS，和自己的JS，无关的不加载

## 改造思路

1. 保留react-create-app默认的入口，js为src/index.js，使用模板public/index.html生成index.html

2. 为src/entries/下的每个子目录，创建一个同名的入口，入口js为src/entries/{entry-name}/index.js, 使用模板public/{entry-name}.html生成{entry-name}.html

按照这个思路我们需要把webpack配置的entry项改为多入口，配置插件HtmlWebpackPlugin为每个entry生成对应的HTML

**注意事项：**

- output.filename需要修改为按入口名生成bundle.js
- optimization.runtimeChunk需要修改为single，这样不同的入口会共用同一个runtime.js而不是每个生成一个
- 默认webpack runtime代码会内嵌到HTML，可以在.env文件中设置INLINE_RUNTIME_CHUNK=false来禁用，或者通过代码删除InlineChunkHtmlPlugin(一个内嵌html的插件)
- devServerConfig.historyApiFallback中disableDotRule设置为true，要不然开发服务器会把/xxx.html的请求重定向到/xxx上，无法打开入口html

## 代码实现

- craco.config.js

```ts
const path = require('path');
const fs = require('fs');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const InlineChunkHtmlPlugin = require('react-dev-utils/InlineChunkHtmlPlugin');
const { getThemeVariables } = require('antd/dist/theme');


function configureWebpack(webpackConfig, {env, paths}) {
    const isEnvDevelopment = env === 'development';
    const isEnvProduction = env === 'production';
    //配置HtmlWebpackPlugin用来产生一个独立的HTML
    function mkHtmlWebpackPlugin(chunks, filename, template) {
        return new HtmlWebpackPlugin({
            inject: true,
            template: template || paths.appHtml,
            chunks,
            filename,
            ...isEnvProduction ? {
                minify: {
                    removeComments: true,
                    collapseWhitespace: true,
                    removeRedundantAttributes: true,
                    useShortDoctype: true,
                    removeEmptyAttributes: true,
                    removeStyleLinkTypeAttributes: true,
                    keepClosingSlash: true,
                    minifyJS: true,
                    minifyCSS: true,
                    minifyURLs: true,
                }
            } : undefined
        });
    }

    //遍历src/entries为所有子目录创建一个webpack入口，并配置对应的HtmlWebpackPlugin
    const entriesDir = path.join(paths.appSrc, 'entries');
    const fileNames = fs.readdirSync(entriesDir);
    const entries = {};
    const htmlWebpackPlugins = [];
    fileNames.forEach(fileName => {
        const filePath = path.join(entriesDir, fileName);
        const file = fs.statSync(filePath);
        if(file.isDirectory()){
            entries[fileName] = path.join(filePath, "index.js");
            let template = path.join(paths.appPublic, fileName + ".html");
            if (!fs.existsSync(template))
                template = undefined;

            htmlWebpackPlugins.push(mkHtmlWebpackPlugin([fileName], fileName + ".html", template));
        }
    });

    //main为create-react-app默认创建的入口，保留下来。这样既可以实现原始的单入口，又可以实现多入口
    webpackConfig.entry = {
        main: webpackConfig.entry,
        ...entries
    };

    //覆盖默认的plugins配置
    const defaultHtmlWebpackPluginIndex = webpackConfig.plugins.findIndex(plugin => plugin instanceof HtmlWebpackPlugin);
    webpackConfig.plugins.splice(defaultHtmlWebpackPluginIndex, 1, mkHtmlWebpackPlugin(["main"], "index.html"), ...htmlWebpackPlugins);

    //create-react-app默认用的是一个固定文件名，不适合多入口！改为按入口名生成输出文件名
    if (isEnvDevelopment)
        webpackConfig.output.filename = 'static/js/[name].bundle.js';

    //共用runtime bundle
    webpackConfig.optimization.runtimeChunk = "single";

    // react-scripts默认在生产环境会将runtime chunk内嵌到html中
    // 禁用该行为，使用独立的js
    // 也可以在根目录新建.env文件，设置INLINE_RUNTIME_CHUNK=false来禁用
    // 不过配置入口太多了，不方便管理，直接这里用代码禁用好了
    const inlineChunkHtmlPluginIndex = webpackConfig.plugins.findIndex(plugin => plugin instanceof InlineChunkHtmlPlugin);
    if (inlineChunkHtmlPluginIndex >= 0)
        webpackConfig.plugins.slice(inlineChunkHtmlPluginIndex, 1);
    //找到oneof rules
    const oneOfRule = webpackConfig.module.rules.find(rule => rule.oneOf);
    const sassRuleIndex = oneOfRule.oneOf.findIndex(
        rule => rule.test && rule.test.toString().includes("scss|sass")
    );
    const lessUse = [...oneOfRule.oneOf[sassRuleIndex].use];
    lessUse[lessUse.length - 1] = {
        loader: require.resolve("less-loader"),
        options: {
            lessOptions: {
                sourceMap: isEnvDevelopment,
                modifyVars: {
                    ...getThemeVariables({
                        compact: false,
                    }),
                    'primary-color': '#fa8c16',
                    'link-color': '#fa8c16',
                    'layout-header-background': '#fa8c16',
                    'border-radius-base': '2px',
                },
                javascriptEnabled: true,
            },
        }
    };
    oneOfRule.oneOf.splice(sassRuleIndex, 0, {
        exclude: /\.module\.(less)$/,
        test:  /\.less$/,
        use: lessUse
    });
    return webpackConfig;
}

function configureDevServer(devServerConfig, { env, paths, proxy, allowedHost }) {
    devServerConfig.historyApiFallback = {
        disableDotRule: true, //禁用，否则当访问/xxx.html时服务器会自动去掉.html重写url为/xxx
        index: paths.publicUrlOrPath,
        verbose: true,
        // rewrites: [
        //     { from: /^\/seller-center/, to: '/seller-center.html' },
        // ]
    };
    return devServerConfig;
}

module.exports = {
    devServer: configureDevServer,
    webpack: {
        configure: configureWebpack,
    },
};
```

