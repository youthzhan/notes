# webpack 知识点

## 一、常见的loader
样式处理：
**style-loader**: 将css添加到DOM的内联样式标签style里
**css-loader** :允许将css文件通过require的方式引入，并返回css代码
**less-loader**: 处理less
**sass-loader**: 处理sass
**postcss-loader**: 用postcss来处理CSS
autoprefixer-loader: 处理CSS3属性前缀，已被弃用，建议直接使用postcss
文件处理：
**file-loader**: 分发文件到output目录并返回相对路径
**url-loader**: 和file-loader类似，但是当文件小于设定的limit时可以返回一个Data Url
**html-minify-loade**r: 压缩HTMLbabel-loader :用babel来转换ES6文件到ES

代码例子🌰
```js
 config.module.rules.push(
      {
        test: /\.module\.(sa|sc)ss$/,
        use: [
          {
            loader: MiniCssExtractPlugin.loader,
            options: {
              hmr: devMode,
            },
          },
          {
            loader: 'css-loader',
            options: {
              modules: true, // 启用CSS Module
            },
          },
          {
            loader: 'sass-loader',
            options: {
              sassOptions: {
                logger: {
                  warn: (message, { deprecation }) => {
                    // 屏蔽告警，需要注意迁移
                  },
                },
              },
            },
          },
        ],
      },
      // 处理非CSS Module的SCSS文件（文件名不包含.module.scss）
      {
        test: /\.(sa|sc)ss$/,
        exclude: /\.module\.(sa|sc)ss$/,
        use: [
          {
            loader: MiniCssExtractPlugin.loader,
            options: {
              hmr: devMode,
            },
          },
          {
            loader: 'css-loader',
            options: {
              modules: false, // 禁用CSS Module
            },
          },
          {
            loader: 'sass-loader',
            options: {
              sassOptions: {
                logger: {
                  warn: (message, { deprecation }) => {
                    // 屏蔽告警，需要注意迁移
                  },
                },
              },
            },
          },
        ],
      },
    );

    config.module.rules.forEach((rule, idx) => {
      if (rule.test instanceof RegExp && rule.test.test('.png')) {
        // eslint-disable-next-line no-param-reassign
        rule.use[0].options.limit = 1024 * 60;
      }

      // 删掉tea中默认svg处理配置，方便自定义配置
      if (rule.test instanceof RegExp && rule.test.test('.svg')) {
        config.module.rules.splice(idx, 1);
      }
    });

    /**
     * 处理 svg
     * 1. react组件内部可直接当成组件引入svg
     * 2. css|sass|scss|less中不引入为组件，引入为url
     */
    config.module.rules.push({
      test: /\.svg$/,
      oneOf: [
        // 如果从 JavaScript/TypeScript 文件中导入
        {
          issuer: /\.[jt]sx?$/,
          use: [
            {
              loader: '@svgr/webpack',
              options: {
                icon: true,
                prettier: false,
                dimensions: false,
              },
            },
          ],
        },
        // 如果从 CSS/SCSS 文件中导入
        {
          issuer: /\.(css|scss|sass|less)$/,
          use: [
            {
              loader: 'url-loader',
              options: {
                limit: 20480,
                name: 'mps.[hash].[ext]',
              },
            },
            // 可以「链式」修改 SVG，最终生成新的SVG文件或内联字符串，常用于「去色、改色、加 class、删属性」等批量处理。
            'svg-transform-loader',
            {
              loader: 'svgo-loader', // svgo-loader 是 Webpack 的 SVGO 压缩管道，把任何进来的 SVG 先瘦身（删冗余、合并路径、压缩属性）
              options: {
                plugins: [{ removeTitle: true }, { convertStyleToAttrs: true }],
              },
            },
          ],
        },
        // 特殊逻辑单独处理
        {
          include: path.resolve(__dirname, 'src/assets/images/workflow'),
          use: [
            {
              loader: 'url-loader',
              options: {
                limit: 20480,
                name: 'mps.[hash].[ext]',
              },
            },
            'svg-transform-loader',
            {
              loader: 'svgo-loader',
              options: {
                plugins: [{ removeTitle: true }, { convertStyleToAttrs: true }],
              },
            },
          ],
        },
      ],
    });
```
Loader的执行顺序是从右到左，从下到上的链式调用。在你的配置中：
> Sass文件 → sass-loader → css-loader → MiniCssExtractPlugin.loader 
1. sass-loader：先将.scss文件编译为CSS
2. css-loader：处理CSS中的依赖关系（如@import）
3. MiniCssExtractPlugin.loader：将CSS提取到独立文件

