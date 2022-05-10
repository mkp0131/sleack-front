# sleack 클론

- mysql 설정: config/config.js
- npx sequelize db:create
- npm run dev (테이블 만들기)
- npx sequelize db:seed:all
- npm run dev (데이터 넣기)

```
npm i react@17.0.2 react-dom@17.0.2
npm i typescript
npm i @types/react @types/react-dom -D
```

```
npm i -D eslint prettier eslint-config-prettier eslint-plugin-prettier
```

- .prettierrc, .eslintrc, tsconfig.json 생성

```json
// .prettierrc
{
  "printWidth": 120, // 한줄에 MAX 120자
  "tabWidth": 2, // space 2칸
  "singleQuote": true, // 호따음표
  "trailingComma": "all", // 객체뒤에 , 붙이겠다
  "semi": true // ; 항상 붙이겠다.
}
```

```json
// .eslintrc
{
  "extends": ["plugin:prettier/recommended", "react-app"] // 프리티어 우선으로 설정
}
```

```json
// tsconfig.json
{
  "compilerOptions": {
    "esModuleInterop": true, // import * as React from "react"; 대신에 import React from "react"; 사용가능
    "sourceMap": true, // 소스맵
    "lib": ["ES2020", "DOM"], // 최신문법
    "jsx": "react", // 리액트
    "module": "esnext", // 최신모듈
    "moduleResolution": "Node", //  module을 node 가 해석
    "target": "es5", // 변환하는 타겟
    "strict": true, // 타입체크엄격하게
    "resolveJsonModule": true, // json 파일 import 가능
    "baseUrl": ".", // import 기본설정
    "paths": {
      // 경로를 변수로 저장
      "@hooks/*": ["hooks/*"],
      "@components/*": ["components/*"],
      "@layouts/*": ["layouts/*"],
      "@pages/*": ["pages/*"],
      "@utils/*": ["utils/*"],
      "@typings/*": ["typings/*"]
    }
  }
}
```

```js
// webpack.config.ts
import path from 'path';
import ReactRefreshWebpackPlugin from '@pmmmwh/react-refresh-webpack-plugin';
import webpack from 'webpack';
import ForkTsCheckerWebpackPlugin from 'fork-ts-checker-webpack-plugin';
import { BundleAnalyzerPlugin } from 'webpack-bundle-analyzer';

const isDevelopment = process.env.NODE_ENV !== 'production';

const config: webpack.Configuration = {
  name: 'sleact',
  mode: isDevelopment ? 'development' : 'production',
  devtool: !isDevelopment ? 'hidden-source-map' : 'eval',
  resolve: {
    extensions: ['.js', '.jsx', '.ts', '.tsx', '.json'],
    alias: {
      '@hooks': path.resolve(__dirname, 'hooks'),
      '@components': path.resolve(__dirname, 'components'),
      '@layouts': path.resolve(__dirname, 'layouts'),
      '@pages': path.resolve(__dirname, 'pages'),
      '@utils': path.resolve(__dirname, 'utils'),
      '@typings': path.resolve(__dirname, 'typings'),
    },
  },
  entry: {
    app: './client',
  },
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        loader: 'babel-loader',
        options: {
          presets: [
            [
              '@babel/preset-env',
              {
                targets: {
                  browsers: ['last 2 chrome versions'],
                },
                debug: isDevelopment,
              },
            ],
            '@babel/preset-react',
            '@babel/preset-typescript',
          ],
          env: {
            development: {
              plugins: [
                ['@emotion', { sourceMap: true }],
                require.resolve('react-refresh/babel'),
              ],
            },
            production: {
              plugins: ['@emotion'],
            },
          },
        },
        exclude: path.join(__dirname, 'node_modules'),
      },
      {
        test: /\.css?$/,
        use: ['style-loader', 'css-loader'],
      },
    ],
  },
  plugins: [
    new ForkTsCheckerWebpackPlugin({
      async: false,
      // eslint: {
      //   files: "./src/**/*",
      // },
    }),
    new webpack.EnvironmentPlugin({
      NODE_ENV: isDevelopment
        ? 'development'
        : 'production',
    }),
  ],
  output: {
    path: path.join(__dirname, 'dist'),
    filename: '[name].js',
    publicPath: '/dist/',
  },
  devServer: {
    historyApiFallback: true, // react router
    port: 3090,
    publicPath: '/dist/',
    proxy: {
      '/api/': {
        target: 'http://localhost:3095',
        changeOrigin: true,
      },
    },
  },
};

if (isDevelopment && config.plugins) {
  config.plugins.push(
    new webpack.HotModuleReplacementPlugin()
  );
  config.plugins.push(new ReactRefreshWebpackPlugin());
  config.plugins.push(
    new BundleAnalyzerPlugin({
      analyzerMode: 'server',
      openAnalyzer: true,
    })
  );
}
if (!isDevelopment && config.plugins) {
  config.plugins.push(
    new webpack.LoaderOptionsPlugin({ minimize: true })
  );
  config.plugins.push(
    new BundleAnalyzerPlugin({ analyzerMode: 'static' })
  );
}

export default config;
```

- tsconfig-for-webpack-config.json 파일 생성

```js
{
  "compilerOptions": {
    "module": "commonjs",
    "moduleResolution": "Node",
    "target": "es5",
    "esModuleInterop": true
  }
}
```

- package.json

```
"dev": "cross-env TS_NODE_PROJECT=\"tsconfig-for-webpack-config.json\" webpack serve --env development",
"build": "cross-env NODE_ENV=production TS_NODE_PROJECT=\"tsconfig-for-webpack-config.json\" webpack",
```

### 코드 스플리팅

```
npm i @loadable/component
```

- loadable 로 컴포넌트를 감싸준다.

```js
import loadable from '@loadable/component';

const Home = loadable(() => import('pages/Home'));
const NotFound = loadable(() => import('pages/NotFound'));

export default function Router() {
  return (
    <BrowserRouter basename={process.env.PUBLIC_URL}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  );
}
```
