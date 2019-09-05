---
layout: post
title:  "Android File存储"
date:   2016-05-20
categories: [android]
---

# Android File存储

> Android系统为我们提供了多种途径可以实现持久化储存，本专题主要讨论常用5种存储方案之一的File存储。  

### 基本概念

**内存（memory）**

**内部存储（InternalStorage）**

**外部存储（ExternalStorage）**

首先我们得搞清楚这三个概念，内存，我们在英文中称作memory，内部存储，我们称为InternalStorage，外部存储我们称为ExternalStorage。

### Android系统提供的持久化方案大致有如下五种

* 网络
* File
* Shared Preference
* ContentProvider
* SQLite

其中网络、Shared Preference、ContentProvider、SQLite都是我们经常使用的，但不是我们本专题的的重点，本专题的重点是File存储，File存储在我们日常开发中经常使用（显示的或隐式的），但也是我们比较容易忽略的知识点，Android很多框架都是基于它实现的，所以掌握好File存储对学习Android系统有很大帮助。

### Android 文件结构

![](/static/img/posts/3ee21be9-418b-4684-8992-b406b97337c7.png)

上图就是Android文件的结构图了，这里给大家介绍一下`/data`、`/mnt`、`/storage`这三个文件夹（因为这涉及到我们接下来的内容），`/data`这个文件夹下中有一个app文件夹是用来存放我们安装应用的apk文件的，同时下面还有一个data目录这个就是我们的内部存储空间了（sharedpreference、数据库文件等都是存在这里的），不同的应用用以包命名的文件夹作区分。mnt和storage就是我们的外部存储空间了，mnt下只是一个storage的链接，真正空间位置还是storage。

### 内部存储和外部存储

在进入主题之前我得明白，所有 Android 设备都有两个文件存储区域：“内部”和“外部”存储。这些名称在 Android 早期产生，当时大多数设备都提供内置的非易失性内存（内部存储），以及移动存储介质，比如微型 SD 卡（外部存储）。一些设备将永久性存储空间划分为“内部”和“外部”分区，即便没有移动存储介质，也始终有两个存储空间，并且无论外部存储设备是否可移动，API 的行为均一致。下面我们来看一下相对于程序来说它们各自有什么区别呢？

* 内部存储：什么是内部存储呢？可以这样理解，除了应用程序本身其他程序无法读取（Root情况不考虑），同时还具有如下性质：

    * 它始终可用（在足够空间下）
    * 当用户卸载您的应用时，系统会从内部存储中移除您的应用的所有文件

* 外部存储

    * 它并非始终可用，因为用户可采用 USB - 存储设备的形式装载外部存储，并在某些情况下会从设备中将其移除。
    * 它是全局可读的，因此此处保存的文件可能不受您控制地被读取。
    * 当用户卸载您的应用时，只有在您通过 getExternalFilesDir() 将您的应用的文件保存在目录中时，系统才会从此处移除您的应用的文件。

### 常见使用内部存储方法

```java
public class Context{
/**
    * 作用：可以方便地再手机中创建文件，并返回文件输出流，用于对文件做写入操作。
    * 
    * @param name 用于指定文件名称，不能包含路径分隔符“/”，如果文件不存在，Android会自动创建它。
    *            创建的文件保存在/data/data/<package name>/files/目录中
    * @param mode mode取值：
    *            MODE_APPEND    私有（只有创建此文件的程序能够使用，其他应用程序不能访问），在原有内容基础上增加数据
    *            MODE_PRIVATE  私有，每次打开文件都会覆盖原来的内容
    * @return
    */
public abstract FileOutputStream openFileOutput(String name, int mode);
public abstract File getFilesDir(); //获取文件系统的绝对路径:/data/data/<package name>/files/
String[] fileList (); //当前应用内部存储路径下的所有文件名
public abstract FileInputStream openFileInput(String name);//与openFileOutput对应
public abstract File getDir(String name, int mode);//在应用程序的数据文件下获取或创建name对应的子目录
public abstract File getCacheDir();//方法可以获得当前的手机自带的存储空间中的当前包文件的路径:/data/data/<package name>/cache/
...
}
```

### 常见使用外部部存储方法

```java
public class Context{
    /**
      *
      * @param type The type of files directory to return. May be {@code null}
    *            for the root of the files directory or one of the following
    *            constants for a subdirectory:
    *            {@link android.os.Environment#DIRECTORY_MUSIC}, //mnt/sdcard/android/data/包名/files/music
    *            {@link android.os.Environment#DIRECTORY_PODCASTS}, 
    *            {@link android.os.Environment#DIRECTORY_RINGTONES},
    *            {@link android.os.Environment#DIRECTORY_ALARMS},
    *            {@link android.os.Environment#DIRECTORY_NOTIFICATIONS},
    *            {@link android.os.Environment#DIRECTORY_PICTURES}, or
    *            {@link android.os.Environment#DIRECTORY_MOVIES}.
    */
public abstract File getExternalFilesDir(@Nullable String type);

public abstract File getExternalCacheDir();//mnt/sdcard/android/data/包名/cache/
...
}
```

```java
public class Environment{
public static String getExternalStorageState();//获取shared/external storage的状态
public static File getExternalStorageDirectory();//返回mnt/sdcard/
...
}
```
Context和Environment都可以用来操作外部存储空间，那么这两这有什么区别呢？一般Context所操作的是私有空间即：`/mnt/sdcard/android/data/包名/`，而Environment操作的是共有空间：`/mnt/sdcard/`。

### 最后福利

```java
public class ExternalStorageHelper {

    // 判断SD卡是否被挂载
    public static boolean isSDCardMounted() {
        // return Environment.getExternalStorageState().equals("mounted");
        return Environment.getExternalStorageState().equals(
                Environment.MEDIA_MOUNTED);
    }

    // 获取SD卡的根目录
    public static String getSDCardBaseDir() {
        if (isSDCardMounted()) {
            return Environment.getExternalStorageDirectory().getAbsolutePath();
        }
        return null;
    }

    // 获取SD卡的完整空间大小，返回MB
    public static long getSDCardSize() {
        if (isSDCardMounted()) {
            StatFs fs = new StatFs(getSDCardBaseDir());
            long count = fs.getBlockCountLong();
            long size = fs.getBlockSizeLong();
            return count * size / 1024 / 1024;
        }
        return 0;
    }

    // 获取SD卡的剩余空间大小
    public static long getSDCardFreeSize() {
        if (isSDCardMounted()) {
            StatFs fs = new StatFs(getSDCardBaseDir());
            long count = fs.getFreeBlocksLong();
            long size = fs.getBlockSizeLong();
            return count * size / 1024 / 1024;
        }
        return 0;
    }

    // 获取SD卡的可用空间大小
    public static long getSDCardAvailableSize() {
        if (isSDCardMounted()) {
            StatFs fs = new StatFs(getSDCardBaseDir());
            long count = fs.getAvailableBlocksLong();
            long size = fs.getBlockSizeLong();
            return count * size / 1024 / 1024;
        }
        return 0;
    }

    // 往SD卡的公有目录下保存文件
    public static boolean saveFileToSDCardPublicDir(byte[] data, String type,
            String fileName) {
        BufferedOutputStream bos = null;
        if (isSDCardMounted()) {
            File file = Environment.getExternalStoragePublicDirectory(type);
            try {
                bos = new BufferedOutputStream(new FileOutputStream(new File(
                        file, fileName)));
                bos.write(data);
                bos.flush();
                return true;
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                try {
                    bos.close();
                } catch (IOException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
            }
        }
        return false;
    }

    // 往SD卡的自定义目录下保存文件
    public static boolean saveFileToSDCardCustomDir(byte[] data, String dir,
            String fileName) {
        BufferedOutputStream bos = null;
        if (isSDCardMounted()) {
            File file = new File(getSDCardBaseDir() + File.separator + dir);
            if (!file.exists()) {
                file.mkdirs();// 递归创建自定义目录
            }
            try {
                bos = new BufferedOutputStream(new FileOutputStream(new File(
                        file, fileName)));
                bos.write(data);
                bos.flush();
                return true;
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                try {
                    bos.close();
                } catch (IOException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
            }
        }
        return false;
    }

    // 往SD卡的私有Files目录下保存文件
    public static boolean saveFileToSDCardPrivateFilesDir(byte[] data,
            String type, String fileName, Context context) {
        BufferedOutputStream bos = null;
        if (isSDCardMounted()) {
            File file = context.getExternalFilesDir(type);
            try {
                bos = new BufferedOutputStream(new FileOutputStream(new File(
                        file, fileName)));
                bos.write(data);
                bos.flush();
                return true;
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                try {
                    bos.close();
                } catch (IOException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
            }
        }
        return false;
    }

    // 往SD卡的私有Cache目录下保存文件
    public static boolean saveFileToSDCardPrivateCacheDir(byte[] data,
            String fileName, Context context) {
        BufferedOutputStream bos = null;
        if (isSDCardMounted()) {
            File file = context.getExternalCacheDir();
            try {
                bos = new BufferedOutputStream(new FileOutputStream(new File(
                        file, fileName)));
                bos.write(data);
                bos.flush();
                return true;
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                try {
                    bos.close();
                } catch (IOException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
            }
        }
        return false;
    }

    // 保存bitmap图片到SDCard的私有Cache目录
    public static boolean saveBitmapToSDCardPrivateCacheDir(Bitmap bitmap,
            String fileName, Context context) {
        if (isSDCardMounted()) {
            BufferedOutputStream bos = null;
            // 获取私有的Cache缓存目录
            File file = context.getExternalCacheDir();

            try {
                bos = new BufferedOutputStream(new FileOutputStream(new File(
                        file, fileName)));
                if (fileName != null
                        && (fileName.contains(".png") || fileName
                                .contains(".PNG"))) {
                    bitmap.compress(Bitmap.CompressFormat.PNG, 100, bos);
                } else {
                    bitmap.compress(Bitmap.CompressFormat.JPEG, 100, bos);
                }
                bos.flush();
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                if (bos != null) {
                    try {
                        bos.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }
            return true;
        } else {
            return false;
        }
    }

    // 从SD卡获取文件
    public static byte[] loadFileFromSDCard(String fileDir) {
        BufferedInputStream bis = null;
        ByteArrayOutputStream baos = new ByteArrayOutputStream();

        try {
            bis = new BufferedInputStream(
                    new FileInputStream(new File(fileDir)));
            byte[] buffer = new byte[8 * 1024];
            int c = 0;
            while ((c = bis.read(buffer)) != -1) {
                baos.write(buffer, 0, c);
                baos.flush();
            }
            return baos.toByteArray();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                baos.close();
                bis.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return null;
    }

    // 从SDCard中寻找指定目录下的文件，返回Bitmap
    public Bitmap loadBitmapFromSDCard(String filePath) {
        byte[] data = loadFileFromSDCard(filePath);
        if (data != null) {
            Bitmap bm = BitmapFactory.decodeByteArray(data, 0, data.length);
            if (bm != null) {
                return bm;
            }
        }
        return null;
    }

    // 获取SD卡公有目录的路径
    public static String getSDCardPublicDir(String type) {
        return Environment.getExternalStoragePublicDirectory(type).toString();
    }

    // 获取SD卡私有Cache目录的路径
    public static String getSDCardPrivateCacheDir(Context context) {
        return context.getExternalCacheDir().getAbsolutePath();
    }

    // 获取SD卡私有Files目录的路径
    public static String getSDCardPrivateFilesDir(Context context, String type) {
        return context.getExternalFilesDir(type).getAbsolutePath();
    }

    public static boolean isFileExist(String filePath) {
        File file = new File(filePath);
        return file.isFile();
    }

    // 从sdcard中删除文件
    public static boolean removeFileFromSDCard(String filePath) {
        File file = new File(filePath);
        if (file.exists()) {
            try {
                file.delete();
                return true;
            } catch (Exception e) {
                return false;
            }
        } else {
            return false;
        }
    }
}
```