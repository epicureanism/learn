多語系環境設定筆記
- [.net core](#net-core)
  - [Resx files](#resx-files)
  - [startup.cs](#startupcs)
  - [Enum to read resx](#enum-to-read-resx)
    - [Enum converter](#enum-converter)
    - [Resource Naming in Resx files](#resource-naming-in-resx-files)
- [Table i18n](#table-i18n)
  - [Separate translation table approach](#separate-translation-table-approach)
  - [Ref](#ref)


# .net core
What we want are
1. using resource files to define contents of different languages
2. setup the .net core env to enable language selection in each request
3. the enum can also use the resx files to display different languages of each items.

## Resx files
* To make the Resx can be shared between different projects, make them public by
  * Right click on the resx file to setup the property of "Custom tool" as "PublicResXFileCodeGenerator"
  * two lines in the resx.cs must be replaced the keywords of "internal" with "public"
```c#
//REPLACE "internal" with "public"
public class Common {        
    // ... 略        
               
    //REPLACE "internal" with "public"
    [global::System.ComponentModel.EditorBrowsableAttribute(global::System.ComponentModel.EditorBrowsableState.Advanced)]
    public static global::System.Resources.ResourceManager ResourceManager {
```
## startup.cs
* ①to set default language and supported languages using "RequestLocalizationOptions"
* ②to set how to handle language selection of each request in middleware of "UseRequestLocalization", the default providers in the orders of:
    1. QueryStringRequestCultureProvider
    1. CookieRequestCultureProvider
    1. AcceptLanguageHeaderRequestCultureProvider
```c#
public void ConfigureServices(IServiceCollection services)
{
    //******* ① Setup default language and supported languages *********/
    #region Middleware after UseRouting, before UseEndpoint
    var supportedCultures = getSupportedCulture();
    var defaultCulture = getDefaultCulture();
    var localizationOptions = new RequestLocalizationOptions()
        .SetDefaultCulture(defaultCulture)
        .AddSupportedCultures(supportedCultures)
        .AddSupportedUICultures(supportedCultures);

    //******* ② this line will handle language selection of each request *********/
    app.UseRequestLocalization(localizationOptions);
    #endregion
}

private string getDefaultCulture()
{
    return GeConfigManager.LanguageSettings.DefaultLanguage;
}
//The supported languaes can be stored as a static IEnumerable object
private string[] getSupportedCulture()
{
    var rs = new List<string>(); 
    foreach (var lang in GeConfigManager.LanguageSettings.SupportedLanguages)
    {
        rs.Add(lang.LanguageCode);
    }

    return rs.ToArray();
}
```


## Enum to read resx
### Enum converter
This enum converter is a enum decorator to convert enum members as multilingual KVP
> [Full implementation](https://gist.github.com/epicureanism/878a2c6af9ca56211a5a7f552558c1d0#file-enumconverter-cs)
```c#
/// <summary>
/// 列舉型別與JSON之間的互相轉換，轉換之結果具多語系效果。
/// 多語系之翻譯應定義於resx 檔案中。
/// Enum 的 JSON 格式為KVP型式。
/// </summary>
/// <typeparam name="T"></typeparam>
public class EnumConverter<T> : JsonConverter where T : Enum
{
    ...skipped...

    /// <summary>
    /// Enum to JSON.
    /// 並支援flag 型式的enum type.
    /// </summary>
    /// <param name="writer"></param>
    /// <param name="value"></param>
    /// <param name="serializer"></param>
    public override void WriteJson(JsonWriter writer, object value, JsonSerializer serializer)
    {
        ...skipped...
    }

    private static bool IsFlagEnum()
    {
        return Attribute.IsDefined(typeof(T), typeof(FlagsAttribute));
    }

    /// <summary>
    /// JSON to Enum. 
    /// { "key": "EnumKey", "value": "Enum value in i18n" }
    /// </summary>
    /// <param name="reader"></param>
    /// <param name="objectType"></param>
    /// <param name="existingValue"></param>
    /// <param name="serializer"></param>
    /// <returns></returns>
    /// <exception cref="JsonException"></exception>
    public override object ReadJson(JsonReader reader, Type objectType, object existingValue, JsonSerializer serializer)
    {
        ...skipped...
    }

    private object ToEnumObject(Dictionary<T, string> dictionary)
    {
        ...skipped...
    }

    /// <summary>
    /// 是dictionary，且Key 的型態為 Enum 才可轉換
    /// </summary>
    /// <param name="objectType"></param>
    /// <returns></returns>
    public override bool CanConvert(Type objectType)
    {
        ...skipped...
    }

}
```
To decorate the Enum
```c#
//To decorate the Enum
[JsonConverter(typeof(EnumConverter<AccountStatus>))]
public enum AccountStatus
{        
    Default,
    Normal,
    Suspend,
    Lock,
    Close
}
```
Expected JSON output according above decoration
```js
"account_status": {
    "key": "Default",
    "value": "Default"
    //in Chinese (if resx was setup correctly)
    //"value": "預設" 
}
```

### Resource Naming in Resx files
```
{EnumType}_{EnumMember}
Ex.
AccountStatus_Default
AccountStatus_Normal
AccountStatus_Suspend
```
----------------



# Table i18n
* For more detail, please refer to [my slides](https://www.slideshare.net/epicureanism/an-approach-to-design-net-core-translation-tables)
## [Separate translation table approach](https://medium.com/walkin/database-internationalization-i18n-localization-l10n-design-patterns-94ff372375c6)
Assumimg the product_type and products table are considered i18n tables.

- products
  - id
  - name
  - product_type_id
  
- productc_translations
  - id
  - name_translation
  - language_id
  - product_id
  
- product_type
  - id
  - name
  
- product_type_translation
  - id
  - name_translation
  - language_id
  - product_type_id
  
- languages
  - language_code
  - name
  - name_in_native
  - date_format
  - currency

## Ref
  - [RFC-4646](https://www.ietf.org/rfc/rfc4646.txt)
  - [code list on wiki](https://zh.wikipedia.org/wiki/%E5%8C%BA%E5%9F%9F%E8%AE%BE%E7%BD%AE)
  - [Language and native names](https://codex.wordpress.org/zh-tw:%E5%A4%9A%E5%9C%8B%E8%AA%9E%E8%A8%80%E6%89%8B%E5%86%8A)
  - [Currency - ISO 4217](https://en.wikipedia.org/wiki/ISO_4217)
  - [Currency Symbols](https://www.xe.com/symbols.php)
