# TextBoxLimited.NET

This is a component for ASP.NET WebForms which is a text field with the ability to limit the number of characters entered.

The component was written in Visual Basic .NET using JavaScript injections.

## Usage

Register the component in ASP.NET Pages:

```asp
<%@ Register Namespace="Nemiro.Web.UI" Assembly="TextBoxLimited" TagPrefix="nas" %>
```

or in **web.config**:

```xml
<pages>
  <controls>
    <add namespace="Nemiro.Web.UI" assembly="TextBoxLimited" tagPrefix="nas" />
  </controls>
</pages>
```

Place a component on the page:

```asp
<nas:TextBoxLimited ID="TextBoxLimited1" runat="server">Hello World!</nas:TextBoxLimited>
```

## Download

> [!WARNING]
> Author does NOT guarantee the functionality of the presented binary files.
> Author is NOT responsible for any damage that may occur when running or using the presented binary files.

**:floppy_disk: [TextBoxLimited v.1.1](textboxlimited_v11.zip)**
