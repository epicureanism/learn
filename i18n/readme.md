# startup.cs
* to set default language and supported languages using "RequestLocalizationOptions"
* to set how to handle language selection of each request in middleware of "UseRequestLocalization", the default providers in the orders of:
    1. QueryStringRequestCultureProvider
    1. CookieRequestCultureProvider
    1. AcceptLanguageHeaderRequestCultureProvider
```c#
public void ConfigureServices(IServiceCollection services)
{
    #region Middleware after UseRouting, before UseEndpoint
    var supportedCultures = getSupportedCulture();
    var defaultCulture = getDefaultCulture();
    var localizationOptions = new RequestLocalizationOptions()
        .SetDefaultCulture(defaultCulture)
        .AddSupportedCultures(supportedCultures)
        .AddSupportedUICultures(supportedCultures);

    //*******IMPORTANT: handle language selection of each request *********/
    app.UseRequestLocalization(localizationOptions);
    #endregion
}

private string getDefaultCulture()
{
    return GeConfigManager.LanguageSettings.DefaultLanguage;
}

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

# resx files
* two lines must be replaced keywords of "internal" with "public"
```c#
//REPLACE "internal" with "public"
public class Common {        
    // ... 略        
               
    //REPLACE "internal" with "public"
    [global::System.ComponentModel.EditorBrowsableAttribute(global::System.ComponentModel.EditorBrowsableState.Advanced)]
    public static global::System.Resources.ResourceManager ResourceManager {
```