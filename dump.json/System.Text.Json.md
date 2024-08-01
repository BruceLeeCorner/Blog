# JsonConverter

## ColorJsonConverter

```c#
public class ColorJsonConverter : JsonConverter<System.Windows.Media.Color>
{
    public override System.Windows.Media.Color Read(ref Utf8JsonReader reader, Type typeToConvert, JsonSerializerOptions options)
    {
        return (System.Windows.Media.Color)System.Windows.Media.ColorConverter
            .ConvertFromString(reader.GetString());
    }

    public override void Write(Utf8JsonWriter writer, System.Windows.Media.Color value, JsonSerializerOptions options)
    {
        writer.WriteRawValue($"\"#{value.A:X2}{value.R:X2}{value.G:X2}{value.B:X2}\"");
    }
}
```



# Utf8JsonWriter

可以直接手写Json字符串，但是比较复杂，如 $"kjkjdosjojdsojodjsoj{Name}"，采用Utf8JsonWriter类，可以较为轻松的构建一个Json字符串。如JsonConverter的`void Write(Utf8JsonWriter writer, System.Windows.Media.Color value, JsonSerializerOptions options)`,可以写成`void Write(StringBuilder builder, System.Windows.Media.Color value, JsonSerializerOptions options)`,理论上这是可行的，但是builder太复杂了，对程序员不友好，所以采用了Utf8JsonWriter。

Utf8JsonWriter常用在JsonConverter中。



>Utf8JsonWriter的作用是逐步构造出一个合法的Json字符串，具有很高的性能。

> Indented 缩进
>
> SkipValidation 是否检查Json字符串合法
>
> WriteRawValue的SkipValidation一般设置成true比较好，也就是不检查。

```c#
var options = new JsonWriterOptions
{
    Indented = true,
    SkipValidation = false
};

using var stream = new MemoryStream();
using var writer = new Utf8JsonWriter(stream, options);

writer.WriteStartArray();

writer.WriteStartObject();
writer.WritePropertyName("Name");
writer.WriteStringValue("Jack");
writer.WritePropertyName("Age");
writer.WriteNumberValue(18);
writer.WritePropertyName("Gender");
writer.WriteBooleanValue(true);
writer.WritePropertyName("Mail");
writer.WriteNullValue();
writer.WriteEndObject();

writer.WriteStartObject();
writer.WritePropertyName("Name");
writer.WriteStringValue("Mary");
writer.WritePropertyName("Age");
writer.WriteNumberValue(17);
writer.WritePropertyName("Gender");
writer.WriteBooleanValue(false);
writer.WritePropertyName("Mail");
writer.WriteNullValue();
writer.WriteEndObject();

writer.WriteEndArray();

writer.Flush();
string json = Encoding.UTF8.GetString(stream.ToArray());
```

```json
[
  {
    "Name": "Jack",
    "Age": 18,
    "Gender": true,
    "Mail": null
  },
  {
    "Name": "Mary",
    "Age": 17,
    "Gender": false,
    "Mail": null
  }
]
```



---



```c#
var options = new JsonWriterOptions
{
    Indented = true,
    SkipValidation = false
};

using var stream = new MemoryStream();
using var writer = new Utf8JsonWriter(stream, options);

writer.WriteStartObject();
writer.WritePropertyName("Name");
writer.WriteStringValue("Jack");
writer.WritePropertyName("Skills");
writer.WriteStartArray();
writer.WriteStringValue("C#");
writer.WriteStringValue("Lua");
writer.WriteEndArray();
writer.WriteEndObject();

writer.Flush();
string json = Encoding.UTF8.GetString(stream.ToArray());
```

```json
{
  "Name": "Jack",
  "Skills": [
    "C#",
    "Lua"
  ]
}
```



---

> WriteBooleanValue,WriteStringValue,WriteNumberValue,WriteNullValue写入一个基元类型，WriteBoolean,WriteString,WriteNumber,WriteNull相当于WritePropertyName + Write...Value。WriteRawValue写入原生字符串，慎用。
>

Utf8JsonWriter每执行一次Write，其相应的Json字符串就会增长。使用WriteRawValue(string rawJson)时，要注意 ① 当前Utf8JsonWriter对应的Json字符串，保证rawJson续拼上去，是合法的Json格式；② 后续调用其他Write，续拼后同样是合法的Json格式。  如每次都会加个`,`,如果rawJson末尾写了`,`,则会导致最终的Json字符串格式错误。

如下实例，下面的Write序列执行完成，得到的并不是一个完成Json字符串。

```c#
writer.WriteStartObject();
writer.WriteNumber("temp",26);
writer.WritePropertyName("date");
writer.WriteEndObject();
```

```json
{
  "temp": 26,
  "date": 
}
```

可以用WriteRawValue补齐。

```c#
writer.WriteStartObject();
writer.WriteNumber("temp",26);
writer.WritePropertyName("date");
writer.WriteRawValue('"' + DateTimeOffset.UtcNow.ToString("yyyy-MM-dd HH:mm:ss") + '"');
writer.WriteEndObject();
```

```json
{
  "temp": 26,
  "date": "2024-07-29 14:50:54"
}
```





