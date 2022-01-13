Skip to content
Search or jump to…
Pull requests
Issues
Marketplace
Explore
 
@yezi12138 
yezi12138
/
track
Private
Code
Issues
Pull requests
Actions
Projects
Security
Insights
Settings
track/track/
Latest commit
yeyongqin 第一次提交
ffac160
8 days ago
Git stats
 History
Files
Type
Name
Latest commit message
Commit time
. .
libs
第一次提交
8 days ago
tests
第一次提交
8 days ago
README.md
第一次提交
8 days ago
package-lock.json
第一次提交
8 days ago
package.json
第一次提交
8 days ago
track.min.js
第一次提交
8 days ago
tsconfig.json
第一次提交
8 days ago
webpack.config.js
第一次提交
8 days ago
README.md
多端埋点介绍（快应用/小程序）
快应用源码放在src/scripts/libs/track，小程序源码放在plugins/track， 需要在app页面引用，并且初始化一些环境配置

～/track/index.js 主函数，包括一些环境的配置

～/track/track.config.js 埋点的配置

～/track/track.min.js 埋点插件的代码

部分字段定义
utrace：用来判断用户唯一性的字段，即不管机型如何，用户始终保持着唯一的值。（可以通过外部传入utrace改变内部用户的值），在快应用是使用设备唯一码进行加密，在小程序则是使用openid进行加密

pid：用户登陆之后的凭证，这个可以作为运营分析用户行为的一个依据。

Track方法
更多方法可以查看git的定义，都有备注描述

默认的生命周期字段
可以在环境配置registerEnv方法中修改lifecyclesEnum或者appcyclesEnum

/**
 * 默认的生命周期枚举
 */
export interface LifecyclesEnum {
  onInit?: string
  onReady?: string
  onShow?: string
  onHide?: string
  onBackPress?: string
  onDestroy?: string
}

/**
 * 全局生命周期枚举
 */
export interface AppcyclesEnum {
  onCreate?: string
  onDestroy?: string
}
前期准备
注意！要使用插件，必须先配置环境（为了适配多个开发环境），调用Track.registerEnv(config)

环境配置如下：

/**
 * 根据配置生成的环境工具类
 */
export interface EnvUtils {
  componentExec: ComponentExec
  pageExec: PageExec
  elExec?: ElExec
  lifecyclesEnum?: LifecyclesEnum
  appcyclesEnum?: AppcyclesEnum
  lifecycles?: ObjectT<Fn>
  appcycles?: ObjectT<Fn>
}

/**
 * 组件执行工具
 * @public {function} getName 获取组件名称
 * @public {function} getParent 获取父节点
 */
export interface ComponentExec {
  /**
   * 获取组件名称
   * @param {Context} context 环境作用域
   */
  getName: (context: Context) => string
  /**
   * 获取父节点
   * @param {Context} context 环境作用域
   */
  getParent: (context: Context) => Node
}

/**
 * 元素执行工具
 * @public {function} getDataset 获取元素的数据集
 * @public {function} getParent 获取父节点
 * @public {function} getListener 获取监听器
 */
export interface ElExec {
  /**
   * 获取元素的数据集
   * @param {Node} Node 元素节点
   */
  getDataset: (node: Node) => {}
  /**
   * 获取父节点
   * @param {Node} Node 元素节点
   */
  getParent: (node: Node) => Node
  /**
   * 获取监听器
   * @param {Context} context 环境作用域
   */
  getListener: (context: Context) => Event
  /**
   * 设置元素的检查器
   */
  bindObserver: (context: Context) => any
  /**
  * 解绑元素的检查器
  */
  unbindObserver: (observer: any) => any
}

/**
 * 元素执行工具
 * @public {function} isPage 是否为页面
 * @public {function} getPath 获取当前路径
 */
export interface PageExec {
  /**
   * 是否为页面
   * @param {Context} context 环境作用域
   */
  isPage: (context: Context) => boolean
  /**
   * 获取当前路径
   * @param {Context} context 环境作用域
   */
  getPath: (context: Context) => string
}

/**
 * 默认的生命周期枚举
 */
export interface LifecyclesEnum {
  onInit?: string
  onReady?: string
  onShow?: string
  onHide?: string
  onBackPress?: string
  onDestroy?: string
}

/**
 * 全局生命周期枚举
 */
export interface AppcyclesEnum {
  onCreate?: string
  onDestroy?: string
}
详细配置可以参考各个项目中~/track/index.js文件

初始化插件
必须先调用registerEnv注册环境，然后调用registerUtraceRule注册utrace。如果没有注册utrace，则埋点的数据都不会上报，规定必须存在utrace。

var config = {} // 同前期准备的配置
Track.registerEnv(config)

let track = new Track(trackConfig)

// 注册设备信息
track.registerDevice(function () {
    return new Promise(resolve => {
        resolve(data || {})
    })
})
// 注册utrace
track.registerUtraceRule(function () {
    return new Promise(resolve => {
        resolve('xx')
    })
})

export default track
埋点配置的api
/**
 * track实例化参数
 * @public {string} src 上报数据的地址
 * @public {ObjectT<any>} additionData 额外携带的参数
 * @public {Page[]} pageTracks 页面配置
 * @public {Component} componentTracks 组件配置
 * @public {GlobalLifeCycles} globalLifeCycles 全局生命周期
 * @public {AppLifeCycles} appLifeCycles 应用生命周期
 * @public {ElTracks} elTracks 元素事件
 * @public {boolean} disableSend 是否禁止发送，改为console
 */
export interface Config {
  src?: string
  additionData?: ObjectT<any>
  pageTracks?: Page[]
  componentTracks?: Component
  globalLifeCycles?: GlobalLifeCycles
  appLifeCycles?: AppLifeCycles
  elTracks?: ElTracks
  disableSend?: boolean
}
disableSend:通常用于开发环境，在开发环境会禁止数据的上报，改为输出方式更加方便调试。

additionData:额外的数据，可以指定环境，或者一些版本号之类的。处于初始化设置外，还可以通过setAdditionData方法设置。

页面监听:api
通过配置pageTracks，实现页面的监听

new Track({
	pageTracks: Page[]
})
/**
 * 页面埋点
 * @public {string} path 埋点的页面路径
 * @public { { Method } } methods 方法的埋点
 * @public { { LifeStyle } } lifeCycles 生命周期的埋点
 */
export interface Page {
  path: string
  methods?: ObjectT<Method>
  lifeCycles?: ObjectT<LifeStyle>
  share?: boolean | void
  observer?: ObjectT<{ topNode: string, nodes: ObjectT<{ [propName: string]: Fn }> }>
}

/**
 * 配置的方法
 * @public {string} type 事件名称
 * @public {string} desc 事件描述
 * @public {SendData} data 事件传递的值
 * @public {'before' | 'after'} execTime 埋点触发的时机
 * @public {string} sendTime 发送的时机
 * @public {function} success 成功触发的函数，返回void则不发送
 * @public {function} fail 失败触发的函数，返回void则不发送
 */
export interface Method {
  type?: string
  desc?: string
  data?: SendData[]
  execTime?: ExecTime
  sendTime?: string
  success?: (data: ObjectT<any>) => ObjectT<any> | void
  fail?: (data: ObjectT<any>) => ObjectT<any> | void
}

/**
 * 配置的生命周期
 * @public {boolean} disabled 是否禁止默认的发送机制
 * @public {SendData} data 事件传递的值
 * @public {Fn} success 成功获取数据后的回调
 */
export interface LifeStyle {
  disabled?: boolean
  data?: SendData[]
  success?: (context: Context) => any
}

/**
 * @public {string} key 获取数据的键名
 * @public {string} name 上传数据的键名
 */
export interface SendData {
  key: string
  name: string
}
监听基本页面信息
// 其实默认就是监听全部页面了，如果仅仅是需要监听页面的pv/uv，完全可以不用配置，或者这样简单配置一下
new Track({
	pageTracks: [
		{
			path: '/pages/web/index',
			name: 'H5页面'
		}
	]
})
禁止页面的生命周期数据采集
new Track({
	pageTracks: [
		{
			path: '/pages/web/index',
			name: 'H5页面'
		}
	],
	lifeCycles: {
    onDestroy: {
      disabled: true
    }
  }
})
监听页面的函数
new Track({
	pageTracks: [
	   {
	   		path: '/a',
	   		desc: '页面a',
	   		methods: {
	   			share: {
                   type: 'share',
                   desc: '分享',
                   execTime: 'before',
                   data: [
                       {
                           key: 'id', // 页面的字段名
                           name: 'id' // 上报的字段名
                       }
                   ]
               }
	   		}
	   }
	]
})
对获取的数据进行修改
new Track({
	pageTracks: [
	   {
	   		path: '/a',
	   		desc: '页面a',
	   		methods: {
          // 页面内部的函数名
	   			share: {
                   type: 'share',
                   desc: '分享',
                   execTime: 'before',
                   data: [
                       {
                           key: 'id', // 页面的字段名
                           name: 'xid' // 上报的字段名
                       }
                   ],
            				success: function (data) {
                      	return data.xid += '001'
                    }
               }
	   		}
	   }
	]
})
注意，如果success返回是空，则不上报埋点数据

特殊的key值
$callbackData: 原函数返回值，比如要对原函数fn: () => string进行埋点，则$callbackData就是返回的[string]

$args: 原函数的传参，比如要对原函数fn: (a, b) => void进行埋点，则$args就是[a,b]

new Track({
	pageTracks: [
	   {
	   		path: '/a',
	   		desc: '页面a',
	   		methods: {
          // 页面内部的函数名
	   			share: {
                   type: 'share',
                   desc: '分享',
                   execTime: 'before',
                   data: [
                       {
                           key: '$args[0]', // 页面的字段名
                           name: 'xid' // 上报的字段名
                       }
                   ]
               }
	   		}
	   }
	]
})
组件api
/**
 * 配置的组件
 */
export interface Component {
  [propName: string]: ObjectT<Method>
}

/**
 * 配置的方法
 * @public {string} type 事件名称
 * @public {string} desc 事件描述
 * @public {SendData} data 事件传递的值
 * @public {'before' | 'after'} execTime 埋点触发的时
 * @public {string} sendTime 发送的时机
 * @public {function} success 成功触发的函数，返回void则不发送
 * @public {function} fail 失败触发的函数，返回void则不发送
 */
export interface Method {
  type?: string
  desc?: string
  data?: SendData[]
  execTime?: ExecTime
  sendTime?: string
  success?: (data: ObjectT<any>) => ObjectT<any> | void
  fail?: (data: ObjectT<any>) => ObjectT<any> | void
}
组件其实和页面的配置大致差不多，但是componentTracks的key值一定要对应组件的名称：

快应用中，组件的名称就是**中的name**,也就是create-window
小程序中，组件的名称则是要配置name字段
基本使用
new Track({
		componentTracks: {
		'video-item': {
            pause: {
                type: 'video_play',
                desc: '视频播放',
                execTime: 'before',
                data: [
                    {
                        key: 'hasStart',
                        name: 'hasStart'
                    },
                    {
                        key: 'duration',
                        name: 'duration'
                    }
                ],
                success: function(data) {
                    if (data.hasStart) {
                        delete data.hasStart;
                        return data;
                    } else {
                        return null;
                    }
                }
            }
        },
		}
})
注意，如果success返回是空，则不上报埋点数据

全局的生命周期
/**
 * 全局生命周期埋点
 */
export interface GlobalLifeCycles {
  [propName: string]: (args: {
    track: Track
    path?: string
    currentPage?: ConvertPage | void
    query?: ObjectT<any>
  }) => any
}
其中，字段值则对应上面提到的默认生命周期: [默认生命周期](# 默认的生命周期字段)

基本使用
new Track({
	// 全局页面生命周期
    globalLifeCycles: {
        onInit({ track, path, query }) {
            // 记录一次浏览离开的页面
            track.oncePath = path;
            // 埋点外部拉起快应用
            if (query.g_ref_s) {
                track.setAdditionData({ g_ref_s: query.g_ref_s });
            }
        },
        // // 拓展全局的实体回退事件，hack页面栈问题
        onBackPress() {
            global.$router.setBack();
            return false;
        }
    },
})
应用生命周期
/**
 * 应用生命周期埋点
 */
export type AppLifeCycles = ObjectT<(args: { track: Track }) => any>
其中，字段值则对应上面提到的默认生命周期: [默认生命周期](# 默认的生命周期字段)

默认会自动采集应用生命周期，如果需要定制，则可以覆盖对应的生命周期。

覆盖应用生命周期

new Track({
	// 应用生命周期
    appLifeCycles: {
        onCreate({ track }) {
            track.track({
                path: '',
                type: 'app_open',
                desc: '打开应用',
                data: {
                      et: appTime.et
                },
                 carryDevice: true
           	});
        },
        onDestroy({ track }) {
            track.track({
                path: '',
                type: 'app_close',
                desc: '关闭应用',
                carryDevice: true
            })
        }
    },
})
元素埋点
监听元素的曝光
目前仅支持小程序 配置页面的observer,目前仅支持单个类，也就是说如果在对象observer里面，只能存在一个字段类（后续开发再优化）

dataset：就是在元素上绑定的自定义值,比如

,拿到的值就是{ id: '123' }
曝光的定义是：元素超出了窗口的viewport

new Track({
	pageTracks: [
		{
			path: '/pages/web/index',
			name: 'H5页面',
			observer: {
				'.test': ({ dataset }) => {
					console.log('曝光', dataset)
					return dataset
				}
			}
		}
	]
})
修改曝光上传的接口,修改registerEnv中的bindObserver函数

bindObserver: function ({ track, currentPage, isPage }) {
			let _observer = null
			if (isPage) {
				_observer = wx.createIntersectionObserver(this, {
					observeAll: true
				})
			} else {
				_observer = this.createIntersectionObserver({
					observeAll: true
				})
			}
			let nodes = currentPage.observer
			_observer.relativeToViewport()
			setTimeout(() => {
				nodes && Object.keys(nodes).forEach(key => {
					_observer.observe(key, (res) => {
						if (res.intersectionRatio > 0) {
							let resp = nodes[key]({ dataset: res.dataset })
							track.track({
								type: 'observer',
								desc: '曝光',
								data: resp || {}
							})
						}
					})
				})
			}, 5000)
			return _observer
}
监听元素点击
/**
 * 元素事件埋点
 */
export interface ElTracks {
  [propName: string]: (args: {
    track: Track
    data: ObjectT<any>
    currentPage: ConvertPage | void
    event: Event
  }) => any
}
基本使用
new Track({
	/**
     * 元素监听
     * 元素绑定data-track 比如data-track-[key]="{{ {} }}"
     * key对应下面的字段名，data是数据
     */
    elTracks: {
        /**
         * 运营位
         */
        deploy: function({ data }) {
            return {
                data: {
                    id: data.id
                },
                url: data.url,
                location: data.location,
                name: data.name
            };
        },
        /**
         * 普通区域点击
         */
        area: function({ data }) {
            return {
                area: data.area,
                text: data.text
            };
        }
    }
})
覆盖/新增生命周期
这个功能主要是为了兼容不同环境的差异性，比如在小程序中，计算活跃时间会和快应用存在一定差异

配置registerEnv的apilifecycles

lifecycles(track) {
		return {
		  // 新增生命周期，注册小程序的组件
			mounted: function() {
				if (this.mpType === 'component') {
					track.wrapMethod(this, true)
				}
			},
			// 覆盖原本的销毁生命周期，修正活跃时间
			onDestroy({
				currentPage
			}) {
				let a = this.tid
				console.log('this', currentPage)
				let time = currentPage.time
				time.at += time.lt - time.bt
				return time
			}
		}
}
© 2022 GitHub, Inc.
Terms
Privacy
Security
Status
Docs
Contact GitHub
Pricing
API
Training
Blog
About
Loading complete
