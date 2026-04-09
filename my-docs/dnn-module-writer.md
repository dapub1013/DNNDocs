# DNN Module Writer Skill

Use this skill when asked to create or scaffold a custom DNN (DotNetNuke) module. Gather the following from the user before generating code:

1. **Module name** (e.g., `Acme.Announcements`) — used as namespace prefix and package name
2. **Company/folder name** (e.g., `Acme`) — used in `DesktopModules/{Company}/{ModuleName}/`
3. **What the module does** — so you can design the data model
4. **Framework** — WebForms (default/most compatible), MVC, or SPA
5. **DNN minimum version** — default `09.00.00`

Generate all files listed under the chosen framework section below.

---

## WebForms Module — Full File List

```
DesktopModules/{Company}/{ModuleName}/
├── {ModuleName}.dnn                        ← manifest
├── {ModuleName}.csproj
├── module.css
├── View.ascx + View.ascx.cs
├── Edit.ascx + Edit.ascx.cs
├── Settings.ascx + Settings.ascx.cs
├── {ModuleName}ModuleBase.cs
├── {ModuleName}ModuleSettingsBase.cs
├── Components/
│   ├── {Entity}Info.cs                     ← DAL2 entity
│   ├── {Entity}Controller.cs               ← CRUD
│   └── {ModuleName}Settings.cs             ← settings wrapper
├── Providers/DataProvider/
│   └── install.SqlDataProvider             ← SQL install script
└── App_LocalResources/
    ├── View.ascx.resx
    ├── Edit.ascx.resx
    └── Settings.ascx.resx
```

---

## Manifest Schema (`.dnn` file)

```xml
<?xml version="1.0" encoding="utf-8"?>
<dotnetnuke type="Package" version="8.0">
    <packages>
        <package name="{Company}.{ModuleName}" type="Module" version="1.0.0">
            <friendlyName>{Friendly Module Name}</friendlyName>
            <description>{Description up to 2000 chars}</description>
            <iconFile>MyIcon.png</iconFile>
            <owner>
                <name>{Author Name}</name>
                <organization>{Company}</organization>
                <url>www.example.com</url>
                <email>support@example.com</email>
            </owner>
            <license src="License.txt" />
            <releaseNotes src="ReleaseNotes.txt" />
            <azureCompatible>true</azureCompatible>
            <dependencies>
                <dependency type="coreVersion">09.00.00</dependency>
            </dependencies>
            <components>

                <!-- Module definition: controls, permissions, features -->
                <component type="Module">
                    <desktopModule>
                        <moduleName>{Company}.{ModuleName}</moduleName>
                        <foldername>{Company}/{ModuleName}</foldername>
                        <businessControllerClass>{Company}.Modules.{ModuleName}.Components.{ModuleName}Controller, {Company}.Modules.{ModuleName}</businessControllerClass>
                        <isAdmin>false</isAdmin>
                        <isPremium>false</isPremium>
                        <supportedFeatures>
                            <!-- Add only if implementing the corresponding interface -->
                            <supportedFeature type="Portable" />    <!-- IPortable -->
                            <supportedFeature type="Searchable" />  <!-- ISearchable -->
                            <supportedFeature type="Upgradeable" /> <!-- IUpgradeable -->
                        </supportedFeatures>
                        <moduleDefinition>
                            <friendlyName>{ModuleName}</friendlyName>
                            <defaultCacheTime>0</defaultCacheTime>
                            <moduleControls>
                                <!-- View control: no controlKey = default view -->
                                <moduleControl>
                                    <controlKey />
                                    <controlSrc>DesktopModules/{Company}/{ModuleName}/View.ascx</controlSrc>
                                    <supportsPartialRendering>False</supportsPartialRendering>
                                    <controlTitle>{ModuleName}</controlTitle>
                                    <controlType>View</controlType>
                                    <iconFile />
                                    <helpUrl />
                                </moduleControl>
                                <!-- Edit control -->
                                <moduleControl>
                                    <controlKey>Edit</controlKey>
                                    <controlSrc>DesktopModules/{Company}/{ModuleName}/Edit.ascx</controlSrc>
                                    <supportsPartialRendering>False</supportsPartialRendering>
                                    <controlTitle>Edit</controlTitle>
                                    <controlType>Edit</controlType>
                                    <iconFile />
                                    <helpUrl />
                                </moduleControl>
                                <!-- Settings control: controlKey="Settings" hooks into DNN's module settings panel -->
                                <moduleControl>
                                    <controlKey>Settings</controlKey>
                                    <controlSrc>DesktopModules/{Company}/{ModuleName}/Settings.ascx</controlSrc>
                                    <supportsPartialRendering>False</supportsPartialRendering>
                                    <controlTitle>Settings</controlTitle>
                                    <controlType>Edit</controlType>
                                    <iconFile />
                                    <helpUrl />
                                </moduleControl>
                            </moduleControls>
                            <permissions>
                                <!-- code = module code, key = permission key, name = display name -->
                                <permission code="EDIT" key="EDIT" name="Edit {ModuleName}" />
                            </permissions>
                        </moduleDefinition>
                    </desktopModule>
                </component>

                <!-- Compiled assembly -->
                <component type="Assembly">
                    <assemblies>
                        <assembly>
                            <path>bin</path>
                            <name>{Company}.Modules.{ModuleName}.dll</name>
                        </assembly>
                    </assemblies>
                </component>

                <!-- ASCX, CSS, resx files go in a zip referenced here -->
                <component type="ResourceFile">
                    <resourceFiles>
                        <basePath>DesktopModules/{Company}/{ModuleName}</basePath>
                        <resourceFile>
                            <name>Resources.zip</name>
                        </resourceFile>
                    </resourceFiles>
                </component>

                <!-- SQL install/upgrade scripts -->
                <component type="Script">
                    <scripts>
                        <basePath>DesktopModules/{Company}/{ModuleName}/Providers/DataProvider</basePath>
                        <script type="Install">
                            <path>/</path>
                            <name>01.00.00.SqlDataProvider</name>
                            <version>01.00.00</version>
                        </script>
                        <script type="UnInstall">
                            <path>/</path>
                            <name>uninstall.SqlDataProvider</name>
                            <version>01.00.00</version>
                        </script>
                    </scripts>
                </component>

            </components>
        </package>
    </packages>
</dotnetnuke>
```

### Manifest notes
- `package name` must be globally unique — always prefix with company name.
- `package type` for a module is always `Module`.
- `foldername` maps to `DesktopModules/{foldername}` on disk.
- `controlKey` empty string = default view. `"Settings"` is special: DNN shows it as a tab in the module's settings panel.
- `controlType` values: `View`, `Edit`, `Admin`, `Host`, `Anonymous`.
- Only declare `supportedFeatures` for interfaces you actually implement.
- Script names must follow the pattern `{version}.SqlDataProvider` (e.g., `01.00.00.SqlDataProvider`). `install.SqlDataProvider` runs before all others on first install; `upgrade.SqlDataProvider` runs after all others.
- Use `{databaseOwner}` and `{objectQualifier}` tokens in all SQL scripts — DNN replaces these at install time.

---

## DAL2 Data Layer

### Entity (`Components/{Entity}Info.cs`)

```csharp
using System;
using DotNetNuke.ComponentModel.DataAnnotations;

namespace {Company}.Modules.{ModuleName}.Components
{
    [TableName("{Company}_{ModuleName}_{Entities}")]   // prefix to avoid collisions
    [PrimaryKey("{Entity}Id")]
    [Scope("ModuleId")]                                 // limits queries to the current module instance
    public class {Entity}Info
    {
        public int {Entity}Id { get; set; }
        public int ModuleId { get; set; }

        // your domain fields here

        // Standard DNN audit columns — always include these
        public int CreatedByUserID { get; set; }
        public DateTime CreatedOnDate { get; set; }
        public int LastModifiedByUserID { get; set; }
        public DateTime LastModifiedOnDate { get; set; }
    }
}
```

### Controller (`Components/{Entity}Controller.cs`)

```csharp
using System.Collections.Generic;
using DotNetNuke.Data;

namespace {Company}.Modules.{ModuleName}.Components
{
    public class {Entity}Controller
    {
        public static IEnumerable<{Entity}Info> Get{Entities}(int moduleId)
        {
            using (IDataContext ctx = DataContext.Instance())
            {
                return ctx.GetRepository<{Entity}Info>().Get(moduleId);
            }
        }

        public static {Entity}Info Get{Entity}(int {entity}Id, int moduleId)
        {
            using (IDataContext ctx = DataContext.Instance())
            {
                return ctx.GetRepository<{Entity}Info>().GetById({entity}Id, moduleId);
            }
        }

        public static void Add{Entity}({Entity}Info item, int userId)
        {
            item.CreatedByUserID = userId;
            item.CreatedOnDate = System.DateTime.Now;
            item.LastModifiedByUserID = userId;
            item.LastModifiedOnDate = System.DateTime.Now;
            using (IDataContext ctx = DataContext.Instance())
            {
                ctx.GetRepository<{Entity}Info>().Insert(item);
            }
        }

        public static void Update{Entity}({Entity}Info item, int userId)
        {
            item.LastModifiedByUserID = userId;
            item.LastModifiedOnDate = System.DateTime.Now;
            using (IDataContext ctx = DataContext.Instance())
            {
                ctx.GetRepository<{Entity}Info>().Update(item);
            }
        }

        public static void Delete{Entity}({Entity}Info item)
        {
            using (IDataContext ctx = DataContext.Instance())
            {
                ctx.GetRepository<{Entity}Info>().Delete(item);
            }
        }
    }
}
```

### SQL install script (`Providers/DataProvider/01.00.00.SqlDataProvider`)

```sql
-- Always use {databaseOwner} and {objectQualifier} tokens
CREATE TABLE {databaseOwner}{objectQualifier}{Company}_{ModuleName}_{Entities} (
    [{Entity}Id]            INT IDENTITY(1,1) NOT NULL,
    [ModuleId]              INT NOT NULL,
    -- your domain columns here
    [CreatedByUserID]       INT NULL,
    [CreatedOnDate]         DATETIME NULL,
    [LastModifiedByUserID]  INT NULL,
    [LastModifiedOnDate]    DATETIME NULL,
    CONSTRAINT [PK_{objectQualifier}{Company}_{ModuleName}_{Entities}]
        PRIMARY KEY CLUSTERED ([{Entity}Id] ASC)
)
GO

ALTER TABLE {databaseOwner}{objectQualifier}{Company}_{ModuleName}_{Entities}
    ADD CONSTRAINT [FK_{objectQualifier}{Company}_{ModuleName}_{Entities}_Modules]
    FOREIGN KEY ([ModuleId])
    REFERENCES {databaseOwner}{objectQualifier}Modules ([ModuleID])
    ON DELETE CASCADE
GO
```

```sql
-- uninstall.SqlDataProvider
DROP TABLE {databaseOwner}{objectQualifier}{Company}_{ModuleName}_{Entities}
GO
```

---

## Settings Pattern

### Settings class (`Components/{ModuleName}Settings.cs`)

```csharp
using System.Collections;
using DotNetNuke.Collections;
using DotNetNuke.Common.Utilities;
using DotNetNuke.Entities.Modules;

namespace {Company}.Modules.{ModuleName}.Components
{
    public class {ModuleName}Settings
    {
        private int ModuleId { get; set; }

        // Add your settings as typed properties
        public bool SomeBooleanSetting { get; set; }
        public string SomeStringSetting { get; set; }

        public {ModuleName}Settings(int moduleId)
        {
            ModuleId = moduleId;
            var allSettings = (Hashtable)(new ModuleController()).GetModuleSettings(moduleId);
            SomeBooleanSetting = allSettings.GetValueOrDefault("SomeBooleanSetting", false);
            SomeStringSetting   = allSettings.GetValueOrDefault("SomeStringSetting", string.Empty);
        }

        // Cache settings to avoid repeated DB round-trips
        public static {ModuleName}Settings Get{ModuleName}Settings(int moduleId)
        {
            var cacheKey = "ModuleSettings" + moduleId;
            var settings = ({ModuleName}Settings)DataCache.GetCache(cacheKey);
            if (settings == null)
            {
                settings = new {ModuleName}Settings(moduleId);
                DataCache.SetCache(cacheKey, settings);
            }
            return settings;
        }

        public void SaveSettings()
        {
            var mc = new ModuleController();
            mc.UpdateModuleSetting(ModuleId, "SomeBooleanSetting", SomeBooleanSetting.ToString());
            mc.UpdateModuleSetting(ModuleId, "SomeStringSetting",   SomeStringSetting);
            DataCache.SetCache("ModuleSettings" + ModuleId, this);
        }
    }
}
```

---

## Base Classes

### `{ModuleName}ModuleBase.cs`

```csharp
using DotNetNuke.Entities.Modules;
using DotNetNuke.Security.Permissions;
using {Company}.Modules.{ModuleName}.Components;

namespace {Company}.Modules.{ModuleName}
{
    public class {ModuleName}ModuleBase : PortalModuleBase
    {
        // Shadow the base Settings property with a strongly-typed version
        public new {ModuleName}Settings Settings
            => {ModuleName}Settings.Get{ModuleName}Settings(ModuleId);

        public bool CanEdit
            => ModulePermissionController.HasModulePermission(ModuleConfiguration.ModulePermissions, "EDIT");
    }
}
```

### `{ModuleName}ModuleSettingsBase.cs`

```csharp
using DotNetNuke.Entities.Modules;
using {Company}.Modules.{ModuleName}.Components;

namespace {Company}.Modules.{ModuleName}
{
    public class {ModuleName}ModuleSettingsBase : ModuleSettingsBase
    {
        public new {ModuleName}Settings Settings
            => {ModuleName}Settings.Get{ModuleName}Settings(ModuleId);
    }
}
```

---

## View Control

### `View.ascx`

```aspx
<%@ Control Language="C#" AutoEventWireup="true"
    CodeBehind="View.ascx.cs"
    Inherits="{Company}.Modules.{ModuleName}.View" %>

<div id="{ModuleName}-<%= ModuleId %>">
    <asp:Repeater runat="server" ID="rp{Entities}"
        OnItemDataBound="rp{Entities}_ItemDataBound"
        OnItemCommand="rp{Entities}_ItemCommand">
        <ItemTemplate>
            <!-- render your entity fields here -->
            <asp:LinkButton ID="cmdEdit"   runat="server" CommandName="Edit"   Visible="false" />
            <asp:LinkButton ID="cmdDelete" runat="server" CommandName="Delete" Visible="false" />
        </ItemTemplate>
    </asp:Repeater>
    <asp:LinkButton runat="server" ID="cmdAdd" OnClick="cmdAdd_Click"
        ResourceKey="cmdAdd" CssClass="dnnPrimaryAction" />
</div>
```

### `View.ascx.cs`

```csharp
using System;
using System.Web.UI.WebControls;
using DotNetNuke.Services.Exceptions;
using {Company}.Modules.{ModuleName}.Components;

namespace {Company}.Modules.{ModuleName}
{
    public partial class View : {ModuleName}ModuleBase
    {
        protected void Page_Load(object sender, EventArgs e)
        {
            try
            {
                if (!Page.IsPostBack)
                {
                    rp{Entities}.DataSource = {Entity}Controller.Get{Entities}(ModuleId);
                    rp{Entities}.DataBind();
                }
            }
            catch (Exception exc)
            {
                Exceptions.ProcessModuleLoadException(this, exc);
            }
        }

        protected void cmdAdd_Click(object sender, EventArgs e)
        {
            Response.Redirect(EditUrl("Edit"));
        }

        protected void rp{Entities}_ItemDataBound(object sender, RepeaterItemEventArgs e)
        {
            if (e.Item.ItemType != ListItemType.Item &&
                e.Item.ItemType != ListItemType.AlternatingItem) return;

            var item = ({Entity}Info)e.Item.DataItem;
            var cmdEdit   = e.Item.FindControl("cmdEdit")   as LinkButton;
            var cmdDelete = e.Item.FindControl("cmdDelete") as LinkButton;

            if (cmdEdit != null)
            {
                cmdEdit.Visible = CanEdit;
                // pass the ID back to the Edit control via query string
                cmdEdit.PostBackUrl = EditUrl(string.Empty, string.Empty, "Edit",
                    "{Entity}Id=" + item.{Entity}Id);
            }
            if (cmdDelete != null)
            {
                cmdDelete.CommandArgument = item.{Entity}Id.ToString();
                cmdDelete.Visible = CanEdit;
            }
        }

        protected void rp{Entities}_ItemCommand(object source, RepeaterCommandEventArgs e)
        {
            try
            {
                if (e.CommandName == "Delete")
                {
                    var item = {Entity}Controller.Get{Entity}(
                        int.Parse(e.CommandArgument.ToString()), ModuleId);
                    if (item != null)
                        {Entity}Controller.Delete{Entity}(item);
                }
                Response.Redirect(DotNetNuke.Common.Globals.NavigateURL());
            }
            catch (Exception exc)
            {
                Exceptions.ProcessModuleLoadException(this, exc);
            }
        }
    }
}
```

---

## Edit Control

### `Edit.ascx`

```aspx
<%@ Control Language="C#" AutoEventWireup="true"
    CodeBehind="Edit.ascx.cs"
    Inherits="{Company}.Modules.{ModuleName}.Edit" %>

<!-- Add your form fields here -->

<asp:LinkButton ID="btnSubmit" runat="server" OnClick="btnSubmit_Click"
    ResourceKey="btnSubmit" CssClass="dnnPrimaryAction" />
<asp:LinkButton ID="btnCancel" runat="server" OnClick="btnCancel_Click"
    ResourceKey="btnCancel" CssClass="dnnSecondaryAction" />
```

### `Edit.ascx.cs`

```csharp
using System;
using DotNetNuke.Security;
using DotNetNuke.Services.Exceptions;
using {Company}.Modules.{ModuleName}.Components;

namespace {Company}.Modules.{ModuleName}
{
    public partial class Edit : {ModuleName}ModuleBase
    {
        private int {Entity}Id => Request.Params.GetValueOrDefault("{Entity}Id", -1);

        protected void Page_Load(object sender, EventArgs e)
        {
            try
            {
                if (!Page.IsPostBack && {Entity}Id > 0)
                {
                    var item = {Entity}Controller.Get{Entity}({Entity}Id, ModuleId);
                    if (item == null) return;
                    // populate form fields from item
                }
            }
            catch (Exception exc)
            {
                Exceptions.ProcessModuleLoadException(this, exc);
            }
        }

        protected void btnSubmit_Click(object sender, EventArgs e)
        {
            try
            {
                var security = new PortalSecurity();
                var item = {Entity}Id > 0
                    ? {Entity}Controller.Get{Entity}({Entity}Id, ModuleId)
                    : new {Entity}Info { ModuleId = ModuleId };

                // Always sanitize user input
                // item.SomeTextField = security.InputFilter(txtSomeField.Text,
                //     PortalSecurity.FilterFlag.NoMarkup |
                //     PortalSecurity.FilterFlag.NoSQL    |
                //     PortalSecurity.FilterFlag.NoScripting);

                if ({Entity}Id > 0)
                    {Entity}Controller.Update{Entity}(item, UserId);
                else
                    {Entity}Controller.Add{Entity}(item, UserId);

                Response.Redirect(DotNetNuke.Common.Globals.NavigateURL());
            }
            catch (Exception exc)
            {
                Exceptions.ProcessModuleLoadException(this, exc);
            }
        }

        protected void btnCancel_Click(object sender, EventArgs e)
        {
            Response.Redirect(DotNetNuke.Common.Globals.NavigateURL());
        }
    }
}
```

---

## Settings Control

### `Settings.ascx`

```aspx
<%@ Control Language="C#" AutoEventWireup="true"
    CodeBehind="Settings.ascx.cs"
    Inherits="{Company}.Modules.{ModuleName}.Settings" %>
<%@ Register TagName="label" TagPrefix="dnn" Src="~/controls/labelcontrol.ascx" %>

<fieldset>
    <div class="dnnFormItem">
        <dnn:Label ID="lblSomeSetting" runat="server"
            ResourceKey="lblSomeSetting" ControlName="chkSomeSetting" />
        <asp:CheckBox runat="server" ID="chkSomeSetting" />
    </div>
</fieldset>
```

### `Settings.ascx.cs`

```csharp
using System;
using DotNetNuke.Services.Exceptions;

namespace {Company}.Modules.{ModuleName}
{
    public partial class Settings : {ModuleName}ModuleSettingsBase
    {
        public override void LoadSettings()
        {
            try
            {
                if (!Page.IsPostBack)
                {
                    chkSomeSetting.Checked = Settings.SomeBooleanSetting;
                }
            }
            catch (Exception exc)
            {
                Exceptions.ProcessModuleLoadException(this, exc);
            }
        }

        public override void UpdateSettings()
        {
            try
            {
                Settings.SomeBooleanSetting = chkSomeSetting.Checked;
                Settings.SaveSettings();
            }
            catch (Exception exc)
            {
                Exceptions.ProcessModuleLoadException(this, exc);
            }
        }
    }
}
```

---

## Key DNN APIs — Quick Reference

| API | Purpose |
|---|---|
| `PortalModuleBase` | Base class for all View/Edit controls |
| `ModuleSettingsBase` | Base class for Settings control; override `LoadSettings()` and `UpdateSettings()` |
| `EditUrl("Edit")` | Returns URL that loads the Edit control for this module |
| `EditUrl("", "", "Edit", "Key=Val")` | EditUrl with extra query string params |
| `DotNetNuke.Common.Globals.NavigateURL()` | Returns current page URL (use for cancel/redirect after save) |
| `ModulePermissionController.HasModulePermission(ModuleConfiguration.ModulePermissions, "EDIT")` | Check if current user has EDIT permission |
| `DataContext.Instance().GetRepository<T>()` | DAL2 repository for CRUD |
| `ModuleController().GetModuleSettings(moduleId)` | Load settings hashtable |
| `ModuleController().UpdateModuleSetting(moduleId, key, value)` | Persist a single setting |
| `DataCache.GetCache(key)` / `DataCache.SetCache(key, obj)` | In-process cache |
| `PortalSecurity.InputFilter(input, flags)` | Sanitize user input; use `NoMarkup \| NoSQL \| NoScripting` for text fields |
| `Exceptions.ProcessModuleLoadException(this, exc)` | Standard DNN exception handler for module controls |
| `LocalizeString("Key")` | Get localized string from the control's `.resx` file |
| `Localization.GetString("Key", LocalResourceFile)` | Get localized string with explicit resource file |

### `PortalSecurity.FilterFlag` combinations for input

```csharp
// Plain text (most restrictive — use for user-submitted text content)
PortalSecurity.FilterFlag.NoMarkup | PortalSecurity.FilterFlag.NoSQL |
PortalSecurity.FilterFlag.NoScripting | PortalSecurity.FilterFlag.NoAngleBrackets

// Allow safe HTML (less restrictive — use for admin-entered rich content)
PortalSecurity.FilterFlag.NoSQL | PortalSecurity.FilterFlag.NoScripting
```

---

## MVC Module — Key Differences

For MVC modules, the structure differs from WebForms:

```
DesktopModules/{Company}/{ModuleName}/
├── Controllers/
│   └── {ModuleName}Controller.cs     ← inherits DnnController
├── Models/
│   └── {Entity}.cs
├── Views/
│   ├── {ModuleName}/
│   │   ├── Index.cshtml              ← default view
│   │   └── Edit.cshtml
│   └── Shared/
│       └── _Layout.cshtml
└── {ModuleName}.dnn
```

**Controller** — inherit from `DotNetNuke.Web.Mvc.Framework.Controllers.DnnController`:

```csharp
using DotNetNuke.Web.Mvc.Framework.ActionFilters;
using DotNetNuke.Web.Mvc.Framework.Controllers;

[DnnHandleError]
public class {ModuleName}Controller : DnnController
{
    [HttpGet]
    public ActionResult Index()
    {
        var items = {Entity}Controller.Get{Entities}(ModuleContext.ModuleId);
        return View(items);
    }
}
```

**View** — use `@Dnn` helper for module context:

```cshtml
@model IEnumerable<{Entity}>
@using DotNetNuke.Web.Mvc.Framework.DnnHelper

<div id="Items-@Dnn.ModuleContext.ModuleId">
    @foreach (var item in Model) { ... }
    <a href="@Dnn.Url.EditUrl("Edit")">Add Item</a>
</div>
```

**Manifest** — `controlType` for MVC uses `"View"` but `controlSrc` points to `{Company}/{ModuleName}/{Controller}/{Action}`:

```xml
<moduleControl>
    <controlKey />
    <controlSrc>{Company}/{ModuleName}/{ModuleName}/Index</controlSrc>
    <controlType>View</controlType>
</moduleControl>
```

---

## Checklist Before Generating

- [ ] Namespace: `{Company}.Modules.{ModuleName}`
- [ ] Table prefix: `{Company}_{ModuleName}_` (avoids collisions with other modules)
- [ ] All SQL uses `{databaseOwner}` and `{objectQualifier}` tokens
- [ ] All user text input passed through `PortalSecurity.InputFilter`
- [ ] Edit control checks permissions server-side (don't trust query string alone)
- [ ] Settings class caches via `DataCache`
- [ ] `uninstall.SqlDataProvider` drops all created tables/views/procs
- [ ] `.dnn` manifest version matches first SQL script version (`01.00.00`)
