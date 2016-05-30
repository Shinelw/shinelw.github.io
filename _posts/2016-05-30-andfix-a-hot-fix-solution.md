---
layout:     post
title:      "Android - 热修复方案"
subtitle:   "AndFix - A Hot Fix Solution"
date:       2016-05-29 12:52:56
author:     "Shinelw"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Android
---

AndFix是阿里开源的一种在线bug热修复的方案，当线上应用出现紧急Bug时，无需再重新发版本，通过发送补丁的方式达到修复Bug的功能，相对于之前同样为阿里开源的Dexposed来说，AndFix支持Android2.3-6.0，并且同时支持Dalvik和ART模式。


## 实现思路

AndFix的实现思路就是方法的替换，在native层动态替换方法，通过native代码中hook java层的代码。大致的思路如下图：

![](https://raw.githubusercontent.com/Shinelw/shinelw.github.io/master/assets/principle.png)

## 使用流程

### 1. 引入依赖

```gradle
compile 'com.alipay.euler:andfix:0.4.0@aar'
```

### 2. 初始化PatchManager

在应用的Application中对AndFix进行初始化，加载Patch。

```java
//初始化patchManger
private PatchManager mPatchManager;
mPatchManager = new PatchManager(this);
mPatchManager.init(appVersion);
```

### 3. 加载patch

```java
//加载 patch
mPatchManager.loadPatch();
```

保证尽快的初始化PatchManager和加载patch，在Applicaton.onCreate()中执行，确保patch可以加载修复。

### 4. ApkPatch工具产生diff差异包

AndFix提供了一个ApkPatch的工具，这个工具的主要功能就是比对两个apk，将不同的类（即修复过的类）标识出来，进行更快速的定位修复。

产生差异包的命令行为：

```java
usage: apkpatch -f <new> -t <old> -o <output> -k <keystore> -p <***> -a <alias> -e <***>
 -a,--alias <alias>     keystore entry alias.
 -e,--epassword <***>   keystore entry password.
 -f,--from <loc>        new Apk file path.
 -k,--keystore <loc>    keystore path.
 -n,--name <name>       patch name.
 -o,--out <dir>         output dir.
 -p,--kpassword <***>   keystore password.
 -t,--to <loc>          old Apk file path.
```

### 5. 添加patch进行修复

从服务器拉取patch以后，将patch添加到patchManager中进行修复操作。

```java
mPatchManager.addPatch(patchDir);
```

以上，就完成了整个AndFix的bug修复过程。

## 源码分析

针对于源码的分析过程，大致跟随AndFix整个修复流程来进行。

### 1. PatchManager

```java
	public void init(String appVersion) {	
		//patch路径的初始化
		if (!mPatchDir.exists() && !mPatchDir.mkdirs()){
			Log.e(TAG, "patch dir create error.");
			return;
		} else if (!mPatchDir.isDirectory()){
			mPatchDir.delete();
			return;
		}
		SharedPreferences sp = mContext.getSharedPreferences(SP_NAME,
				Context.MODE_PRIVATE);
		String ver = sp.getString(SP_VERSION, null);
		//针对apk版本的patch处理，如果版本不一致清除，版本一直则加载。
		if (ver == null || !ver.equalsIgnoreCase(appVersion)) {
			//清除patch
			cleanPatch();
			sp.edit().putString(SP_VERSION, appVersion).commit();
		} else {
			//初始化本地的patch
			initPatchs();
		}
	}
	private void initPatchs() {
		File[] files = mPatchDir.listFiles();
		for (File file : files) {
			//添加本地的patch
			addPatch(file);
		}
	}
```

在**init()**方法中，主要对本地的patch进行处理，当apk的版本与patch的版本一致，就加载本地的patch；版本不一致则清除。

接下来我们来看**loadPatch()**的代码实现：

```java
	public void loadPatch() {
		mLoaders.put("*", mContext.getClassLoader());// wildcard
		Set<String> patchNames;
		List<String> classes;
		for (Patch patch : mPatchs) {
			patchNames = patch.getPatchNames();
			for (String patchName : patchNames) {
				classes = patch.getClasses(patchName);
				mAndFixManager.fix(patch.getFile(), mContext.getClassLoader(),
						classes);
			}
		}
	}
```

**loadPatch()**中对本地的patch进行了遍历，获取每个patch的信息，逐一进行**fix()**，其中参数*classes*为patch中配置文件*Patch.MF*的*Patch-Classes*字段对应的所有类。

### 2. AndFixManager

```java
	public synchronized void fix(File file, ClassLoader classLoader,
			List<String> classes) {
		//是否支持	
		if (!mSupport) {
			return;
		}
		//一系列安全检查
		if (!mSecurityChecker.verifyApk(file)) {
			return;
		}
		try {
			File optfile = new File(mOptDir, file.getName());
			boolean saveFingerprint = true;
			if (optfile.exists()) {
				if (mSecurityChecker.verifyOpt(optfile)) {
					saveFingerprint = false;
				} else if (!optfile.delete()) {
					return;
				}
			}
			//获得.apatch补丁中的dex文件
			final DexFile dexFile = DexFile.loadDex(file.getAbsolutePath(),
					optfile.getAbsolutePath(), Context.MODE_PRIVATE);
			if (saveFingerprint) {
				mSecurityChecker.saveOptSig(optfile);
			}
			//类加载器规则：
			//1.类找不到但注解为"com.alipay.euler.andfix",说明是patch中新增的类，返回该类
			//2.类找不到且无注解，抛出classnotfound异常
			//3.类找到，返回该类
			ClassLoader patchClassLoader = new ClassLoader(classLoader) {
				@Override
				protected Class<?> findClass(String className)
						throws ClassNotFoundException {
					Class<?> clazz = dexFile.loadClass(className, this);
					if (clazz == null
							&& className.startsWith("com.alipay.euler.andfix")) {
						return Class.forName(className);
					}
					if (clazz == null) {
						throw new ClassNotFoundException(className);
					}
					return clazz;
				}
			};
			//枚举dex中的所有类，逐一比对，加载需要修复的类
			Enumeration<String> entrys = dexFile.entries();
			Class<?> clazz = null;
			while (entrys.hasMoreElements()) {
				String entry = entrys.nextElement();
				//如果patch中不包含该类，则不需要修复，跳过
				if (classes != null && !classes.contains(entry)) {
					continue;
				}
				//加载需要修复的类
				clazz = dexFile.loadClass(entry, patchClassLoader);
				if (clazz != null) {
					//下一步，修复
					fixClass(clazz, classLoader);
				}
			}
		} catch (IOException e) {
			Log.e(TAG, "pacth", e);
		}
	}
```

**fix()**方法中主要进行的过程就是一系列的安全检查，通过patch配置文件的比对，加载出需要修复的类，然后进行下一步fixClass()的操作。

```java
		private void fixClass(Class<?> clazz, ClassLoader classLoader) {
		//得到类所有公用方法
		Method[] methods = clazz.getDeclaredMethods();
		MethodReplace methodReplace;
		String clz;
		String meth;
		//枚举方式，通过注解找到需要替换的类
		for (Method method : methods) {
			methodReplace = method.getAnnotation(MethodReplace.class);
			if (methodReplace == null)
				continue;
			clz = methodReplace.clazz();
			meth = methodReplace.method();
			if (!isEmpty(clz) && !isEmpty(meth)) {
				//需要替换的类，执行下一步
				replaceMethod(classLoader, clz, meth, method);
			}
		}
	}
```

**fixClass()**方法中进行的过程就是从需要修复的类中定位到需要修复的方法。

```java
	private void replaceMethod(ClassLoader classLoader, String clz,
			String meth, Method method) {
		try {
			String key = clz + "@" + classLoader.toString();
			//得到原apk中要替换的类
			Class<?> clazz = mFixedClass.get(key);
			//如果该类还没有加载
			if (clazz == null) {
				Class<?> clzz = classLoader.loadClass(clz);
				// 初始化该类
				clazz = AndFix.initTargetClass(clzz);
			}
			//该类已初始化完成，得到要替换的方法
			if (clazz != null) {
				mFixedClass.put(key, clazz);
				Method src = clazz.getDeclaredMethod(meth,
						method.getParameterTypes());
				//开始进入native层进行方法的替换
				AndFix.addReplaceMethod(src, method);
			}
		} catch (Exception e) {
			Log.e(TAG, "replaceMethod", e);
		}
	}
```

### 3. AndFix

AndFix是Java层进行方法替换的核心类，在该类中提供了Native层的接口，加载了**andfix.cpp**，主要进行了Native层的初始化，以及目标修复类的替换工作。

```java
		public static void addReplaceMethod(Method src, Method dest) {
		try {
			//jni方法
			replaceMethod(src, dest);
			initFields(dest.getDeclaringClass());
		} catch (Throwable e) {
			Log.e(TAG, "addReplaceMethod", e);
		}
	}
```

**addReplaceMethod()**中替换方法正式从Java层进入Native层。

**initFields()**方法：

```java
private static void initFields(Class<?> clazz) {
		Field[] srcFields = clazz.getDeclaredFields();
		for (Field srcField : srcFields) {
			Log.d(TAG, "modify " + clazz.getName() + "." + srcField.getName()
					+ " flag:");
			//native层方法
			setFieldFlag(srcField);
		}
	}
```

该方法将被替换类的方法全部改成public， 具体操作也在native层实现。

### 4. Native层分析

针对Dalvik和ART，在Native层中有不同的处理。首先，*AndFixManager*类中对系统是否支持进行了判断。

```java
	mSupport = Compat.isSupport();
```

**Compat.isSupport()**的处理如下：

```java
	public static synchronized boolean isSupport() {
		if (isChecked)
			return isSupport;
		isChecked = true;
		//不支持云os
		if (!isYunOS() && AndFix.setup() && isSupportSDKVersion()) {
			isSupport = true;
		}
		if (inBlackList()) {
			isSupport = false;
		}
		return isSupport;
	}
```

**AndFix.setup()**处理如下：

```java
public static boolean setup() {
		try {
			final String vmVersion = System.getProperty("java.vm.version");
			//判断是否是art
			boolean isArt = vmVersion != null && vmVersion.startsWith("2");
			int apilevel = Build.VERSION.SDK_INT;
			//native层方法
			return setup(isArt, apilevel);
		} catch (Exception e) {
			Log.e(TAG, "setup", e);
			return false;
		}
}
```	

进入Native层，针对于art和dalvik，有各自的处理方法。我们来看**andfix.cpp**中的*setup（）*方法。

**setup()方法：**

```c++
static jboolean setup(JNIEnv* env, jclass clazz, jboolean isart,
		jint apilevel) {
	isArt = isart;
	LOGD("vm is: %s , apilevel is: %i", (isArt ? "art" : "dalvik"),
			(int )apilevel);
	if (isArt) {
		return art_setup(env, (int) apilevel);
	} else {
		return dalvik_setup(env, (int) apilevel);
	}
}
```

根据ART和Dalvik的不同，*setup()*方法分别进入不同的方法执行，这里先分析*Dalvik*部分。

进入*dalvik_setup()*方法中，位于*dalvik_method_replace.cpp*中，如下：


```c++
extern jboolean __attribute__ ((visibility ("hidden"))) dalvik_setup(
		JNIEnv* env, int apilevel) {
	//Davik虚拟机实现 是在libdvm.so中
	//dlopen()方法以指定模式打开动态链接库，RTLD_NOW立即打开
	void* dvm_hand = dlopen("libdvm.so", RTLD_NOW);
	if (dvm_hand) {
		//dvm_dlsym:通过句柄和连接符名称获取函数或变量名
		dvmDecodeIndirectRef_fnPtr = dvm_dlsym(dvm_hand,
				apilevel > 10 ?
						"_Z20dvmDecodeIndirectRefP6ThreadP8_jobject" :
						"dvmDecodeIndirectRef");
		if (!dvmDecodeIndirectRef_fnPtr) {
			return JNI_FALSE;
		}
		dvmThreadSelf_fnPtr = dvm_dlsym(dvm_hand,
				apilevel > 10 ? "_Z13dvmThreadSelfv" : "dvmThreadSelf");
		if (!dvmThreadSelf_fnPtr) {
			return JNI_FALSE;
		}
		jclass clazz = env->FindClass("java/lang/reflect/Method");
		jClassMethod = env->GetMethodID(clazz, "getDeclaringClass",
						"()Ljava/lang/Class;");
		return JNI_TRUE;
	} else {
		return JNI_FALSE;
	}
}
```

该方法进行的操作主要是打开运行dalvik虚拟机的libdvm.so，得到dvmDecodeIndirectRef_fnPtr、dvmThreadSelf_fnPtr函数，下面将用到这两个函数获取类对象。

接下来我们进入整个AndFix最核心的**dalvik_replaceMethod()**方法中，在其中进行了对类方法指针的替换，真正实现对方法的替换。

```c++
extern void __attribute__ ((visibility ("hidden"))) dalvik_replaceMethod(
		JNIEnv* env, jobject src, jobject dest) {
	//clazz为被替换的类
	jobject clazz = env->CallObjectMethod(dest, jClassMethod);
	//clz 为被替换的类对象
	ClassObject* clz = (ClassObject*) dvmDecodeIndirectRef_fnPtr(
			dvmThreadSelf_fnPtr(), clazz);
	//将类状态设置为装载完毕
	clz->status = CLASS_INITIALIZED;
	//得到指向新类的指针
	Method* meth = (Method*) env->FromReflectedMethod(src);
	//得到指向需要修复的目标类的指针
	Method* target = (Method*) env->FromReflectedMethod(dest);
	
	//新类指向目标类，实现类的替换
	meth->clazz = target->clazz;
	meth->accessFlags |= ACC_PUBLIC;
	meth->methodIndex = target->methodIndex;
	meth->jniArgInfo = target->jniArgInfo;
	meth->registersSize = target->registersSize;
	meth->outsSize = target->outsSize;
	meth->insSize = target->insSize;
	meth->prototype = target->prototype;
	meth->insns = target->insns;
	meth->nativeFunc = target->nativeFunc;
}
```

至此，AndFix的整个方法替换流程已经结束。

总体的过程总结如下：
1. 初始化patch管理器，加载补丁；
2. 检查手机是否支持，判断ART、Dalvik；
3. 进行md5，指纹的安全检查
4. 验证补丁的配置，通过patch-classes字段得到要替换的所有类
5. 通过注解从类中得到具体要替换的方法
6. 修改方法的访问权限为public
7. 得到指向新类和被替换类的指针，将新类指向目标类，完成方法的替换。

概括一句话就是，Java层进行补丁的加载，手机支持，安全检查，得到替换类等一系列前期准备工作；Native层进行类权限的修改、类的替换等核心功能。

## ApkPatch工具原理分析

讲完了AndFix整个的替换流程，最后一节我们来分析一下生成diff差异包补丁的原理。

使用jd-gui逆向打开apkpatch.jar文件，整个jar的代码树如下图：

![]()















