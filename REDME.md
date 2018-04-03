在原生Android上集成ReactNative
* ⚠️注意细节
    * yarn工具如果start命令提示不存在，则将其更新到最新版本
    * 端口默认使用8081，若被其他应用占用，使用yarn start —port=8088，使用8088端口（lsof -i:8081可以查看占用端口的应用）
    * 在项目的build.gradle设置maven库时，注意好相对位置，如果是项目被移到/android文件夹下则使用：
        * maven {
        *     url "$rootDir/../node_modules/react-native/android"
        * }
    * 若在同一文件夹下则使用
        * maven {
        *     url "$rootDir/node_modules/react-native/android"
        * }
    * 使用设备时，调用 adb reverse tcp:8088 tcp:8088指向指定端口
    * 在使用yarn start启动server后，在设备中要调用Dev Setting（摇晃手机），在里面设置主机ip和端口（保证在同一wifi下，且手机不要有代理）
* 正常步骤
    * 在根文件夹创建文件package.json
{
    "name": "MyReactNativeApp",
    "version": "0.0.1",
    "private": true,
    "scripts": {
        "start": "node node_modules/react-native/local-cli/cli.js start"
    },
}

    * 调用yarn add react-native和yarn add react@16.3.0添加react和react-native依赖，同时生成node_modules文件夹（注意此时与根目录的相对位置）
    * 在app的build.gradle添加依赖：
        * dependencies {
        *     compile 'com.android.support:appcompat-v7:23.0.1'
        *     ...
        *     compile "com.facebook.react:react-native:+" // From node_modules
        * }
    * 在项目的build.gradle添加maven库：
        *  allprojects {
        *     repositories {
        *         maven {
        *             // All of React Native (JS, Android binaries) is installed from npm，此时的node_modules的相对位置
        *             url "$rootDir/node_modules/react-native/android"
        *         }
        *         ...
        *     }
        *     ...
        * }
    * 在AndroidManifest添加
        * <uses-permission android:name="android.permission.INTERNET" />
        * <activity android:name="com.facebook.react.devsupport.DevSettingsActivity" /> //启动DevSetting时需要
    * 创建index.android.js（注意yarn start时指定需要index.android.js而不是index.js）
        * import React from 'react';
        * import {AppRegistry, StyleSheet, Text, View} from 'react-native';

        * class HelloWorld extends React.Component {
        *   render() {
        *     return (
        *       <View style={styles.container}>
        *         <Text style={styles.hello}>My friend.Is it alright now?I cant believe</Text>
        *       </View>
        *     );
        *   }
        * }
        * var styles = StyleSheet.create({
        *   container: {
        *     flex: 1,
        *     justifyContent: 'center',
        *   },
        *   hello: {
        *     fontSize: 20,
        *     textAlign: 'center',
        *     margin: 10,
        *   },
        * });
    * 创建MyReactActivity（在里面使用ReactRootView渲染）
public class MyReactActivity extends AppCompatActivity implements DefaultHardwareBackBtnHandler{
    private ReactRootView mReactRootView;
    private ReactInstanceManager mReactInstanceManager;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        mReactRootView = new ReactRootView(this);
        mReactInstanceManager = ReactInstanceManager.builder()
                .setApplication(getApplication())
                .setBundleAssetName("index.android.bundle")
                .setJSMainModulePath("index.android")	//注意你创建的是index.js还是index.android.js
                .addPackage(new MainReactPackage())
                .setUseDeveloperSupport(BuildConfig.DEBUG)
                .setInitialLifecycleState(LifecycleState.RESUMED)
                .build();
        // The string here (e.g. "MyReactNativeApp") has to match
        // the string in AppRegistry.registerComponent() in index.js
        mReactRootView.startReactApplication(mReactInstanceManager, "MyReactNativeApp", null);

        setContentView(mReactRootView);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();

        if (mReactInstanceManager != null) {
            mReactInstanceManager.onHostDestroy(this);
        }
    }

    @Override
    protected void onPause() {
        super.onPause();

        if (mReactInstanceManager != null) {
            mReactInstanceManager.onHostPause(this);
        }
    }

    @Override
    protected void onResume() {
        super.onResume();

        if (mReactInstanceManager != null) {
            mReactInstanceManager.onHostResume(this, this);
        }
    }

    @Override
    public void invokeDefaultOnBackPressed() {
        if (mReactInstanceManager != null) {
            mReactInstanceManager.onBackPressed();
        } else {
            super.onBackPressed();
        }
    }
}
    * 使用设备进行调试时，先在src/main下创建一个文件夹assets，然后调用
react-native bundle --platform android --dev false --entry-file index.android.js --bundle-output app/src/main/assets/index.android.bundle --assets-dest app/src/main/res/
生成index.android.bundle
    * 调用yarn start —port=8088，如果8081被占用的话
    * 调用adb reverse tcp:8088 tcp:8088

