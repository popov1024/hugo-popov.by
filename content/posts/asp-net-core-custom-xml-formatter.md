---
title: "ASP.Net Core custom XML formatter with Windows-1251"
date: 2018-04-09
draft: false
---

Возникла задача обработки `XML` в кодировке `Windows-1251`. Штатный форматтер `XML` в `MVC` `ASP.Net Core` умеет выполнять обработку только `UTF-8`. Но так же MVC поддерживает возможность создания собственных форматтеров.

Сам форматтер `CustomXmlInputFormatter.cs`

``` csharp
using Microsoft.AspNetCore.Mvc.Formatters;
using System;
using System.Threading.Tasks;
using System.Text;
using Microsoft.Net.Http.Headers;
using System.IO;
using System.Xml;
using System.Xml.Serialization;

namespace api
{
    public class CustomXmlInputFormatter: TextInputFormatter
    {
        public CustomXmlInputFormatter()
        {
            SupportedMediaTypes.Add(MediaTypeHeaderValue.Parse("application/xml"));

            Encoding.RegisterProvider(CodePagesEncodingProvider.Instance);
            var Windows1251 = CodePagesEncodingProvider.Instance.GetEncoding(1251);
            SupportedEncodings.Add(Windows1251);
            SupportedEncodings.Add(Encoding.UTF8);
        }

        public override async Task<InputFormatterResult> ReadRequestBodyAsync(InputFormatterContext context, Encoding encoding)
        {
            if (context == null)
            {
                throw new ArgumentNullException(nameof(context));
            }

            if (encoding == null)
            {
                throw new ArgumentNullException(nameof(encoding));
            }

            var request = context.HttpContext.Request;

            // Для поддержки Windows-1251 при дессмриализации
            Encoding.RegisterProvider(CodePagesEncodingProvider.Instance);
            using (var streamReader = context.ReaderFactory(request.Body, encoding))
            {
                var type = context.ModelType;

                try
                {
                    var serializer = new XmlSerializer(type);
                    var model = serializer.Deserialize(streamReader);

                    return await InputFormatterResult.SuccessAsync(model);
                }
                catch (Exception)
                {
                    return await InputFormatterResult.FailureAsync();
                }
            }
        }
    }
}
```

Подключение форматтера в `Startup.cs`

``` csharp
services.AddMvc(options =>  
{  
    options.InputFormatters.Add(new CustomXmlInputFormatter());
});
```
