## ASP.NET MVC 5

## Removing "?aspxerrorpath=" query string appended to URLs on 404 redirects with attribute-based routing enabled
When attribute-based routing is enabled in an ASP.NET MVC 5 project, if a 'customErrors' tag is present in the web.config file handling 404 redirection, an annoying query  string ("aspxerrorpath") is appended to the end of the URL after page redirect.  Adding the following code to the Global.asax file will remove the query string:

```void Application_Error(object sender, EventArgs e)
{
    Exception ex = Server.GetLastError();

    if (ex is HttpException && ((HttpException)ex).GetHttpCode() == 404)
    {
        Response.Redirect("/home");
    }
}
```