# DataBinding

## 如何绑定附加依赖属性

### 在XAML中绑定附加属性

在 XAML 中绑定附加属性的时候需要加上括号和类型的命名空间前缀：

```c#
<ListViewItem IsEnabled="{Binding (local:DraggableElement.IsDraggable), RelativeSource={RelativeSource Self}}"
              local:DraggableElement.IsDraggable="True" />
```

对于 WPF 内置的命名空间（`http://schemas.microsoft.com/winfx/2006/xaml/presentation` 命名空间下），是不需要加前缀的。

```c#
<TextBlock x:Name="DemoTextBlock" Grid.Row="1"
           Text="{Binding (Grid.Row), RelativeSource={RelativeSource Self}}" />
```

跟其他的绑定一样，这里并不需要在 `Binding` 后面写 `Path=`，因为 `Binding` 的构造函数中传入的参数就是赋值给 `Path` 的。

### 在C#代码中绑定附加属性

在C#代码中绑定附加属性，需要**使用PropertyPath，而不能使用字符串**！

这段代码`无法工作`,仅演示！

```c#
Binding binding = new Binding("(Grid.Row)")
{
    Source = DemoTextBlock
}
BindingOperations.SetBinding(DemoTextBlock, TextBox.TextProperty, binding);
```

这段代码可以`正常工作`！

```c#
Binding binding = new Binding
{
    Source = DemoTextBlock,
    Path = new PropertyPath(Grid.RowProperty)
}
BindingOperations.SetBinding(DemoTextBlock, TextBox.TextProperty, binding);
```

# Command

# Trigger

```xaml
<ControlTemplate x:Key="SubMunu_ShowOnTop" TargetType="{x:Type MenuItem}">
    <ControlTemplate.Triggers>
        <Trigger Property="IsHighlighted" Value="true">
            <Setter Property="Foreground" Value="white" />
            <Setter TargetName="BtnBottomLine" Property="Stroke" Value="white" />
        </Trigger>
        <DataTrigger Binding="{Binding Path=Selected}" Value="True">
            <Setter Property="Foreground" Value="white" />
            <Setter TargetName="TranBG" Property="Fill" Value="Red" />
        </DataTrigger>
    </ControlTemplate.Triggers>
</ControlTemplate>
```

