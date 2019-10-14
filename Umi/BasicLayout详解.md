
```js
/* 
  基础页面布局，包含了头部导航，侧边栏和通知栏
*/
/* 
  Suspense 处理异步配合lazy使用方法如下
  import React, {lazy, Suspense} from 'react';
  const OtherComponent = lazy(() => import('./OtherComponent'));
  
  function MyComponent() {
    return (
      <Suspense fallback={<div>Loading...</div>}>
        <OtherComponent />
      </Suspense>
    );
  }

*/
import React, { Suspense } from 'react';
import { Layout } from 'antd';
/* 
  react-document-title
  根据不同的路由改变文档的title
*/
import DocumentTitle from 'react-document-title';
import isEqual from 'lodash/isEqual';
/* 
  memoize-one
  这个库的每个实例都缓存了一个结果
  记忆化库
  memoizeOne(resultFn, isEqual)
  接收一个结果函数和一个对比函数，对比函数为空则默认使用===来进行入参的比较。
*/
import memoizeOne from 'memoize-one';
import { connect } from 'dva';
/* 
  react-container-query 
  https://www.npmjs.com/package/react-container-query
  响应组件
  参数 
    query 响应式的断点位置
    props.children 需要是一个返回组件的函数
    <ContainerQuery query={query}>
      {params => (
        <Context.Provider value={this.getContext()}>
          <div className={classNames(params)}>{layout}</div>
        </Context.Provider>
      )}
    </ContainerQuery>
*/
import { ContainerQuery } from 'react-container-query';
/* 
  https://github.com/JedWatson/classnames
  classNames('foo', 'bar'); // => 'foo bar'
  classNames('foo', { bar: true }); // => 'foo bar'
  classNames({ 'foo-bar': true }); // => 'foo-bar'
  classNames({ 'foo-bar': false }); // => ''
  classNames({ foo: true }, { bar: true }); // => 'foo bar'
  classNames({ foo: true, bar: true }); // => 'foo bar'

  // lots of arguments of various types
  classNames('foo', { bar: true, duck: false }, 'baz', { quux: true }); // => 'foo bar baz quux'

  // other falsy values are just ignored
  classNames(null, false, 'bar', undefined, 0, 1, { baz: null }, ''); // => 'bar 1'
*/
import classNames from 'classnames';
/* 
        在路径字符串中使用正则
*/
import pathToRegexp from 'path-to-regexp';
/* 
        添加响应式，根据屏幕大小返回不同组件
*/
import Media from 'react-media';
/* 
        基于 umi-plugin-locale 和 react-intl 实现，用于解决 i18n 问题
        https://umijs.org/zh/plugin/umi-plugin-react.html#locale
        https://github.com/umijs/umi/tree/master/packages/umi-plugin-locale
*/
import { formatMessage } from 'umi/locale';
/* 
        权限组件
*/
import Authorized from '@/utils/Authorized';
import logo from '../assets/logo.svg';
import Footer from './Footer';
import Header from './Header';
import Context from './MenuContext';
import Exception403 from '../pages/Exception/403';
import PageLoading from '@/components/PageLoading';
import SiderMenu from '@/components/SiderMenu';

import styles from './BasicLayout.less';

// lazy load SettingDrawer
const SettingDrawer = React.lazy(() => import('@/components/SettingDrawer'));

const { Content } = Layout;

// 添加 react-container-query 参数，和antd 断点尺寸做搭配
const query = {
  'screen-xs': {
    maxWidth: 575,
  },
  'screen-sm': {
    minWidth: 576,
    maxWidth: 767,
  },
  'screen-md': {
    minWidth: 768,
    maxWidth: 991,
  },
  'screen-lg': {
    minWidth: 992,
    maxWidth: 1199,
  },
  'screen-xl': {
    minWidth: 1200,
    maxWidth: 1599,
  },
  'screen-xxl': {
    minWidth: 1600,
  },
};

class BasicLayout extends React.PureComponent {
  constructor(props) {
    super(props);
    this.getPageTitle = memoizeOne(this.getPageTitle);
    this.matchParamsPath = memoizeOne(this.matchParamsPath, isEqual);
  }

  componentDidMount() {
    const {
      dispatch,
      route: { routes, authority },
    } = this.props;
    dispatch({
      type: 'user/fetchCurrent',
    });
    dispatch({
      type: 'setting/getSetting',
    });
    dispatch({
      type: 'menu/getMenuData',
      payload: { routes, authority },
    });
  }

  componentDidUpdate(preProps) {
    // After changing to phone mode,
    // if collapsed is true, you need to click twice to display
    const { collapsed, isMobile } = this.props;
    if (isMobile && !preProps.isMobile && !collapsed) {
      this.handleMenuCollapse(false);
    }
  }

  getContext() {
    const { location, breadcrumbNameMap } = this.props;
    return {
      location,
      breadcrumbNameMap,
    };
  }

  matchParamsPath = (pathname, breadcrumbNameMap) => {
    const pathKey = Object.keys(breadcrumbNameMap).find(key => pathToRegexp(key).test(pathname));
    return breadcrumbNameMap[pathKey];
  };
  // 根据当前path 的到路由权限的方法
//   getRouterAuthority = (pathname, routeData) => {
//     let routeAuthority = ['noAuthority'];
//     const getAuthority = (key, routes) => {
//       routes.map(route => {
//         // pathToRegexp 将当前路由path转成正则表达式
//         if (route.path && pathToRegexp(route.path).test(key)) {
//           routeAuthority = route.authority;
//         } else if (route.routes) {
//           // 如果含有子路由递归
//           routeAuthority = getAuthority(key, route.routes);
//         }
//         return route;
//       });
//       return routeAuthority;
//     };
//     return getAuthority(pathname, routeData);
//   };

  getPageTitle = (pathname, breadcrumbNameMap) => {
    const currRouterData = this.matchParamsPath(pathname, breadcrumbNameMap);

    if (!currRouterData) {
      return 'Ant Design Pro';
    }
    const pageName = formatMessage({
      id: currRouterData.locale || currRouterData.name,
      defaultMessage: currRouterData.name,
    });

    return `${pageName} - Ant Design Pro`;
  };

  getLayoutStyle = () => {
    const { fixSiderbar, isMobile, collapsed, layout } = this.props;
    if (fixSiderbar && layout !== 'topmenu' && !isMobile) {
      return {
        paddingLeft: collapsed ? '80px' : '256px',
      };
    }
    return null;
  };

  handleMenuCollapse = collapsed => {
    const { dispatch } = this.props;
    dispatch({
      type: 'global/changeLayoutCollapsed',
      payload: collapsed,
    });
  };

  renderSettingDrawer = () => {
    // Do not render SettingDrawer in production
    // unless it is deployed in preview.pro.ant.design as demo
    if (process.env.NODE_ENV === 'production' && APP_TYPE !== 'site') {
      return null;
    }
    return <SettingDrawer />;
  };

  render() {
    const {
      navTheme,
      layout: PropsLayout,
      children,
      location: { pathname },
      isMobile,
      menuData,
      breadcrumbNameMap,
      // this.props 中包含 route? 这里 routes中包含所有的当前route下的子路由
      route: { routes },
      fixedHeader,
    } = this.props;
    const isTop = PropsLayout === 'topmenu';
    // 路由配置的权限
    const routerConfig = this.getRouterAuthority(pathname, routes);
    const contentStyle = !fixedHeader ? { paddingTop: 0 } : {};
    const layout = (
      <Layout>
        {isTop && !isMobile ? null : (
          <SiderMenu
            logo={logo}
            theme={navTheme}
            onCollapse={this.handleMenuCollapse}
            menuData={menuData}
            isMobile={isMobile}
            {...this.props}
          />
        )}
        <Layout
          style={{
            ...this.getLayoutStyle(),
            minHeight: '100vh',
          }}
        >
          <Header
            menuData={menuData}
            handleMenuCollapse={this.handleMenuCollapse}
            logo={logo}
            isMobile={isMobile}
            {...this.props}
          />
          <Content className={styles.content} style={contentStyle}>
            <Authorized authority={routerConfig} noMatch={<Exception403 />}>
              {children}
            </Authorized>
          </Content>
          <Footer />
        </Layout>
      </Layout>
    );
    return (
      <React.Fragment>
        <DocumentTitle title={this.getPageTitle(pathname, breadcrumbNameMap)}>
          {/* 
            全局响应式断点 在布局最外层添加class 方便给不同的元素添加响应式样式
          */}
          <ContainerQuery query={query}>
            {params => (
              <Context.Provider value={this.getContext()}>
                <div className={classNames(params)}>{layout}</div>
              </Context.Provider>
            )}
          </ContainerQuery>
        </DocumentTitle>
        <Suspense fallback={<PageLoading />}>{this.renderSettingDrawer()}</Suspense>
      </React.Fragment>
    );
  }
}

/* 
  umi
  model 分两类，一是全局 model，二是页面 model。全局 model 存于 /src/models/ 目录，所有页面都可引用；页面 model 不能被其他页面所引用。
*/
export default connect(({ global, setting, menu }) => ({
  collapsed: global.collapsed,
  layout: setting.layout,
  menuData: menu.menuData,
  breadcrumbNameMap: menu.breadcrumbNameMap,
  ...setting,
}))(props => (
  /* 
    < 599 
  */
  <Media query="(max-width: 599px)">
    {isMobile => <BasicLayout {...props} isMobile={isMobile} />}
  </Media>
));


```