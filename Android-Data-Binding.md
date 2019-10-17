## Data-Binding

### Bindable

可对`java-bean`对象实现数据变动后自动触发刷新UI界面得功能。

实现“数据变更触发ui界面”的话，途径主要有两个：

* 继承 `BaseObservable ，使用 Bindable 注解field的getter并且在调用setter的使用使用 OnPropertyChangedCallback#onPropertyChanged

* 使用data-binding library当中提供的诸如 ObservableField<> , ObservableInt作为属性值

使用示例

```java
public class PrivateUser extends BaseObservable{

    private String fistName;
    private String lastName;

    public PrivateUser(String fistName, String lastName) {
        this.age = age;
        this.fistName = fistName;
        this.lastName = lastName;
    }

    @Bindable
    public String getFistName() {
        return fistName;
    }

    public void setFistName(String fistName) {
        this.fistName = fistName;
        
        //需调用此方法来通知UI更新
        notifyPropertyChanged(BR.fistName);
    }

    @Bindable
    public String getLastName() {
        return lastName;
    }

    public void setLasetName(String lasetName) {
        this.lastName = lastName;
        notifyPropertyChanged(BR.lastName);

    }
  }
```


### `BindingAdapter`

可自定义实现XML属性中View属性的set方法。

### BindingConversion

类型转换用

编译器会自动匹配方法

### BindingMethods & BindingMethod


重新命名view属性对应的setter方法名称

注解参数中指定三个关键值，分别是：

type：属性所关联的view的class对象
attribute： 属性的名称（如果是android框架的属性使用“android”作为命名空间，如果是应用本身定义的属性则不需要附带命名空间）
method：对应的setter方法名称

使用示例

```java
@BindingMethods({
       @BindingMethod(type = "android.widget.ImageView",
                      attribute = "android:tint",
                      method = "setImageTintList"),
})
```
