

# MotionLayout

* 原始文档：https://developer.android.com/training/constraint-layout/motionlayout



## 简介

### 概述
* MotionLayout是一种布局，用于管理运动和控件动画
* MotionLayout是ConstraintLayout的子类，依赖于其丰富的布局能力
* MotionLayout填平了布局过渡动画和复杂运行处理之间的鸿沟
    * 通过提供一种混合特征在**属性动画框架**，**过渡管理器**，**约束布局**之间

#### 特征
* 支持给布局属性增加动画，用于在布局之间增加过渡动画
* 支持可搜索(seekable)过渡，用于根据一些条件(触摸输入)立刻在过渡动画中显示任何点
* 支持关键帧，用于完全自定义过渡效果
* 是完全的声明式的，可以在XML中描述任何过渡效果


### 设计考虑
* MotionLayout设计用来移动、缩放、动画那些可用户产生互动的界面元素（像按键和标题栏）
* 应用中的运动并不是简单的、免费的特效，而是可以辅助用户来理解应用正在做的事情


### 开始
#### 操作步骤
1. 添加 ConstraintLayout 依赖，需要2.0版本
```groovy
dependencies {
    implementation 'androidx.constraintlayout:constraintlayout:2.0.0-beta1'
}
```
1. 创建MotionLayout文件
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.motion.widget.MotionLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/motionLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:layoutDescription="@xml/scene_01"
    tools:showPaths="true">

    <View
        android:id="@+id/button"
        android:layout_width="64dp"
        android:layout_height="64dp"
        android:background="@color/colorAccent"
        android:text="Button" />

</androidx.constraintlayout.motion.widget.MotionLayout>
```
1. 创建运动场景


#### 运动场景
* 是一个XML资源文件，包含了对于布局的所有运动描述
* 为了从布局信息中独立出运动信息，每个布局都需要引入一个分离的运动场景
* 布局中通过app:layoutDescription来引用运动场景

##### 例子
```xml
<?xml version="1.0" encoding="utf-8"?>
<MotionScene xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:motion="http://schemas.android.com/apk/res-auto">

    <Transition
        motion:constraintSetStart="@+id/start"
        motion:constraintSetEnd="@+id/end"
        motion:duration="1000">
        <OnSwipe
            motion:touchAnchorId="@+id/button"
            motion:touchAnchorSide="right"
            motion:dragDirection="dragRight" />
    </Transition>

    <ConstraintSet android:id="@+id/start">
        <Constraint
            android:id="@+id/button"
            android:layout_width="64dp"
            android:layout_height="64dp"
            android:layout_marginStart="8dp"
            motion:layout_constraintBottom_toBottomOf="parent"
            motion:layout_constraintStart_toStartOf="parent"
            motion:layout_constraintTop_toTopOf="parent" >
             <CustomAttribute
            motion:attributeName="backgroundColor"
            motion:customColorValue="#D81B60" />
        </Constraint>
    </ConstraintSet>

    <ConstraintSet android:id="@+id/end">
        <Constraint
            android:id="@+id/button"
            android:layout_width="64dp"
            android:layout_height="64dp"
            android:layout_marginEnd="8dp"
            motion:layout_constraintBottom_toBottomOf="parent"
            motion:layout_constraintEnd_toEndOf="parent"
            motion:layout_constraintTop_toTopOf="parent" >
            <CustomAttribute
            motion:attributeName="backgroundColor"
            motion:customColorValue="#9999FF" />
        </Constraint>
    </ConstraintSet>

</MotionScene>
```
说明
* Transition 包含了运动的基础定义
    * motion:constraintSetStart,motion:constraintSetEnd引用运动的开始、结束点
    * 开始、结束点用于ConstraintSet来定义
    * motion:duration 指定运动花费的时间（毫秒数）
* OnSwipe 通过触摸来控制运动
    * motion:touchAnchorId 引用可以点击、拖动的视图
    * motion:touchAnchorSide 指定拖动的锚点（例子中从右边开始）
    * motion:dragDirection 指定拖动的进度方向
* ConstraintSet 定义运动的各种约束信息



### 内插属性
* 在运动场景文件中，ConstraintSet可以包含一些附加属性，用于在过渡动画中进行内插处理
* 包括  
    * alpha  透明度
    * visibility 可见性
    * elevation 
    * rotation, rotationX, rotationY 旋转
    * translationX, translationY, translationZ 平移
    * scaleX, scaleY 缩放


### 自定义属性
* 使用CustomAttribute可以指定属性的一些复杂变换
* 图示
```xml
<Constraint
    android:id="@+id/button" ...>
    <CustomAttribute
        motion:attributeName="backgroundColor"
        motion:customColorValue="#D81B60"/>
</Constraint>
```

#### CustomAttribute属性
* motion:attributeName 必须的，必须要有getter,setter方法
    * backgroundColor必须要有getBackgroundColor()和setBackgroundColor()方法
* 另一个属性必须用以下类型
    * motion:customColorValue 用于颜色值
    * motion:customIntegerValue 用于整型值
    * motion:customFloatValue 用于浮点型值
    * motion:customStringValue 用于字符串值
    * motion:customDimensionValue 用于尺寸值
    * motion:customBooleanValue 用于布尔值

#### 注意
* 使用自定义属性，需要在开始、结束的ConstraintSet元素中都定义


### 运动布局的附加属性
* app:applyMotionScene="boolean" 表示是否应用运动场景，默认为true
* app:showPaths="boolean" 表示是否在运动时，显示运行路径，默认为false
* app:progress="float" 明确特定的过渡进度值，范围：0 -- 1 
* app:currentState="reference" 表明当前的特定约束集
* app:motionDebug 显示附加的调试信息




## 例子
* github上的example应用
* https://github.com/android/views-widgets-samples/tree/master/ConstraintLayoutExamples