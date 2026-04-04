# Telerik to Bootstrap/ASP.NET Migration Skill

Migrates ASP.NET Web Forms applications from Telerik UI controls to standard ASP.NET controls + Bootstrap 5 + vanilla JavaScript. Eliminates commercial Telerik dependency while preserving all functionality.

## When to Use

Use when replacing Telerik Web UI controls in ASP.NET Web Forms (.NET Framework 4.x) applications with free alternatives. Handles markup (.aspx/.ascx), code-behind (.cs), and designer (.designer.cs) files.

## Migration Strategy

Work file by file. For each file:
1. Replace markup tags (Telerik → ASP.NET + Bootstrap)
2. Update code-behind event handlers and type casts
3. Regenerate designer.cs declarations
4. Test compilation

Always preserve the existing data-binding logic and stored procedure calls. Only change the UI layer.

## Control Replacement Reference

### RadScriptManager → ScriptManager
```aspx
<!-- BEFORE -->
<telerik:RadScriptManager ID="RadScriptManager1" runat="server">
    <Scripts>
        <asp:ScriptReference Assembly="Telerik.Web.UI" Name="Telerik.Web.UI.Common.Core.js" />
        <asp:ScriptReference Assembly="Telerik.Web.UI" Name="Telerik.Web.UI.Common.jQuery.js" />
    </Scripts>
</telerik:RadScriptManager>

<!-- AFTER -->
<asp:ScriptManager ID="ScriptManager1" runat="server" EnablePageMethods="true" />
<script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet" />
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
```

**Designer.cs:** `protected global::System.Web.UI.ScriptManager ScriptManager1;`

---

### RadGrid → GridView + Bootstrap table
```aspx
<!-- BEFORE -->
<telerik:RadGrid ID="RadGrid2" runat="server" RenderMode="Lightweight" Width="100%"
    AutoGenerateColumns="False" AllowPaging="True" PageSize="20"
    AllowSorting="true" AllowFilteringByColumn="true"
    OnNeedDataSource="RadGrid2_NeedDataSource"
    OnSelectedIndexChanged="RadGrid2_SelectedIndexChanged"
    OnItemDataBound="RadGrid2_ItemDataBound">
    <ClientSettings EnableRowHoverStyle="true" EnablePostBackOnRowClick="true">
        <Selecting AllowRowSelect="True" />
    </ClientSettings>
    <MasterTableView DataKeyNames="ID" CommandItemDisplay="Top">
        <Columns>
            <telerik:GridBoundColumn DataField="Name" HeaderText="Name" UniqueName="Name" />
            <telerik:GridTemplateColumn HeaderText="Actions" UniqueName="Actions">
                <ItemTemplate>
                    <telerik:RadButton ID="btnEdit" runat="server" Text="Edit" OnClick="btnEdit_Click" />
                </ItemTemplate>
            </telerik:GridTemplateColumn>
            <telerik:GridNumericColumn DataField="Amount" DataType="System.Decimal"
                NumericType="Currency" HeaderText="Amount" />
        </Columns>
        <NoRecordsTemplate>
            <div class="alert alert-info">No records found.</div>
        </NoRecordsTemplate>
    </MasterTableView>
</telerik:RadGrid>

<!-- AFTER -->
<asp:UpdatePanel ID="upGrid" runat="server">
<ContentTemplate>
<asp:GridView ID="gvMain" runat="server" CssClass="table table-striped table-hover"
    AutoGenerateColumns="False" AllowPaging="True" PageSize="20"
    AllowSorting="True" DataKeyNames="ID"
    OnRowDataBound="gvMain_RowDataBound"
    OnSelectedIndexChanged="gvMain_SelectedIndexChanged"
    OnPageIndexChanging="gvMain_PageIndexChanging"
    OnSorting="gvMain_Sorting">
    <HeaderStyle CssClass="table-dark" />
    <SelectedRowStyle CssClass="table-primary" />
    <Columns>
        <asp:BoundField DataField="Name" HeaderText="Name" SortExpression="Name" />
        <asp:TemplateField HeaderText="Actions">
            <ItemTemplate>
                <asp:LinkButton ID="btnEdit" runat="server" Text="Edit"
                    CssClass="btn btn-sm btn-outline-primary" OnClick="btnEdit_Click" />
            </ItemTemplate>
        </asp:TemplateField>
        <asp:BoundField DataField="Amount" HeaderText="Amount" DataFormatString="{0:C}" />
    </Columns>
    <EmptyDataTemplate>
        <div class="alert alert-info">No records found.</div>
    </EmptyDataTemplate>
</asp:GridView>
</ContentTemplate>
</asp:UpdatePanel>
```

**Code-behind translation:**
```csharp
// BEFORE: RadGrid
protected void RadGrid2_NeedDataSource(object sender, GridNeedDataSourceEventArgs e)
{
    (sender as RadGrid).DataSource = GetData();
}
protected void RadGrid2_ItemDataBound(object sender, GridItemEventArgs e)
{
    GridDataItem item = e.Item as GridDataItem;
    if (item != null) { string val = item["ColumnName"].Text; }
}
GridDataItem item = (GridDataItem)RadGrid2.MasterTableView.Items[RadGrid2.SelectedItems[0].ItemIndex];
string value = item["ColumnName"].Text;
RadGrid2.Rebind();

// AFTER: GridView
private void BindGrid()
{
    gvMain.DataSource = GetData();
    gvMain.DataBind();
}
protected void gvMain_RowDataBound(object sender, GridViewRowEventArgs e)
{
    if (e.Row.RowType == DataControlRowType.DataRow)
    {
        string val = DataBinder.Eval(e.Row.DataItem, "ColumnName").ToString();
    }
}
GridViewRow row = gvMain.SelectedRow;
string value = row.Cells[columnIndex].Text;
BindGrid(); // replaces Rebind()
```

**Designer.cs:** `protected global::System.Web.UI.WebControls.GridView gvMain;`

**Grid column type mapping:**
| Telerik | ASP.NET |
|---------|---------|
| `GridBoundColumn` | `BoundField` |
| `GridTemplateColumn` | `TemplateField` |
| `GridNumericColumn` | `BoundField` with `DataFormatString="{0:C}"` |
| `GridCheckBoxColumn` | `CheckBoxField` |
| `GridDateTimeColumn` | `BoundField` with `DataFormatString="{0:d}"` |
| `GridHyperLinkColumn` | `HyperLinkField` |

---

### RadButton → Button/LinkButton + Bootstrap
```aspx
<!-- BEFORE -->
<telerik:RadButton ID="btnSave" runat="server" Text="Save" RenderMode="Lightweight"
    Primary="true" Font-Bold="true" Height="32" Width="140"
    OnClick="btnSave_Click" CausesValidation="true" ValidationGroup="grp1">
    <Icon PrimaryIconCssClass="rbSave" PrimaryIconTop="7px" />
</telerik:RadButton>

<!-- AFTER -->
<asp:Button ID="btnSave" runat="server" Text="Save"
    CssClass="btn btn-primary fw-bold" OnClick="btnSave_Click"
    CausesValidation="true" ValidationGroup="grp1" />
```

**Icon mapping (Telerik → Bootstrap Icons):**
| Telerik Icon | Bootstrap Equivalent |
|-------------|---------------------|
| `rbSave` | `<i class="bi bi-save"></i>` |
| `rbNext` | `<i class="bi bi-arrow-right"></i>` |
| `rbEdit` | `<i class="bi bi-pencil"></i>` |
| `rbCancel` | `<i class="bi bi-x-circle"></i>` |
| `rbMail` | `<i class="bi bi-envelope"></i>` |

For buttons with icons, use LinkButton:
```aspx
<asp:LinkButton ID="btnSave" runat="server" CssClass="btn btn-primary fw-bold"
    OnClick="btnSave_Click">
    <i class="bi bi-save"></i> Save
</asp:LinkButton>
```

**Designer.cs:** `protected global::System.Web.UI.WebControls.Button btnSave;` or `LinkButton`

---

### RadDropDownList / RadComboBox → DropDownList + Bootstrap
```aspx
<!-- BEFORE -->
<telerik:RadDropDownList ID="ddlCountry" runat="server" Width="200"
    DataTextField="CountryName" DataValueField="CountryID"
    OnSelectedIndexChanged="ddlCountry_SelectedIndexChanged" AutoPostBack="true" />

<!-- AFTER -->
<asp:DropDownList ID="ddlCountry" runat="server" CssClass="form-select" Width="200"
    DataTextField="CountryName" DataValueField="CountryID"
    OnSelectedIndexChanged="ddlCountry_SelectedIndexChanged" AutoPostBack="true" />
```

**Code-behind:**
```csharp
// BEFORE
ddlStatus.Items.Remove(ddlStatus.FindItemByText("Paid"));
RadDropDownList ddl = (RadDropDownList)item.FindControl("ddlStatus");

// AFTER
ListItem li = ddlStatus.Items.FindByText("Paid");
if (li != null) ddlStatus.Items.Remove(li);
DropDownList ddl = (DropDownList)item.FindControl("ddlStatus");
```

**Designer.cs:** `protected global::System.Web.UI.WebControls.DropDownList ddlCountry;`

---

### RadTextBox → TextBox + Bootstrap
```aspx
<!-- BEFORE -->
<telerik:RadTextBox ID="txtName" runat="server" Width="200" MaxLength="50" />

<!-- AFTER -->
<asp:TextBox ID="txtName" runat="server" CssClass="form-control" Width="200" MaxLength="50" />
```

**Designer.cs:** `protected global::System.Web.UI.WebControls.TextBox txtName;`

---

### RadNumericTextBox → TextBox + validation
```aspx
<!-- BEFORE -->
<telerik:RadNumericTextBox ID="txtAmount" runat="server" Width="120" Type="Currency" />

<!-- AFTER -->
<asp:TextBox ID="txtAmount" runat="server" CssClass="form-control" Width="120" TextMode="Number" />
```

---

### RadSwitch → CheckBox + Bootstrap
```aspx
<!-- BEFORE -->
<telerik:RadSwitch ID="cbkActive" runat="server" />

<!-- AFTER -->
<div class="form-check form-switch">
    <asp:CheckBox ID="cbkActive" runat="server" CssClass="form-check-input" />
</div>
```

**Code-behind:** `cbkActive.Checked` (same, but Telerik uses `.Checked.Value` for nullable)

**Designer.cs:** `protected global::System.Web.UI.WebControls.CheckBox cbkActive;`

---

### RadDatePicker → TextBox with type="date"
```aspx
<!-- BEFORE -->
<telerik:RadDatePicker ID="dpDateFrom" runat="server" Width="150" />

<!-- AFTER -->
<asp:TextBox ID="dpDateFrom" runat="server" CssClass="form-control" Width="150" TextMode="Date" />
```

**Code-behind:**
```csharp
// BEFORE
DateTime? dt = dpDateFrom.SelectedDate;
dpDateFrom.SelectedDate = DateTime.Now;

// AFTER
DateTime dt;
DateTime.TryParse(dpDateFrom.Text, out dt);
dpDateFrom.Text = DateTime.Now.ToString("yyyy-MM-dd");
```

**Designer.cs:** `protected global::System.Web.UI.WebControls.TextBox dpDateFrom;`

---

### RadLabel → Label
```aspx
<!-- BEFORE -->
<telerik:RadLabel ID="labStatus" runat="server" Text="Active" />

<!-- AFTER -->
<asp:Label ID="labStatus" runat="server" Text="Active" />
```

**Designer.cs:** `protected global::System.Web.UI.WebControls.Label labStatus;`

---

### RadPanelBar → Bootstrap accordion/nav
```aspx
<!-- BEFORE -->
<telerik:RadPanelBar ID="mainMenu" runat="server" RenderMode="Lightweight"
    ExpandMode="MultipleExpandedItems" Skin="Black" PersistStateInCookie="true">
    <Items>
        <telerik:RadPanelItem Text="Users" Font-Bold="true">
            <Items>
                <telerik:RadPanelItem Text="Applicants" NavigateUrl="applicants.aspx"
                    ImageUrl="../images/com013.svg" Visible="false" />
            </Items>
        </telerik:RadPanelItem>
    </Items>
</telerik:RadPanelBar>

<!-- AFTER -->
<div class="accordion" id="mainMenu">
    <div class="accordion-item" id="menuUsers" runat="server">
        <h2 class="accordion-header">
            <button class="accordion-button fw-bold" type="button"
                data-bs-toggle="collapse" data-bs-target="#collapseUsers">Users</button>
        </h2>
        <div id="collapseUsers" class="accordion-collapse collapse show" data-bs-parent="#mainMenu">
            <div class="list-group list-group-flush">
                <a id="navApplicants" runat="server" class="list-group-item list-group-item-action"
                    href="applicants.aspx" visible="false">
                    <img src="../images/com013.svg" width="16" /> Applicants
                </a>
            </div>
        </div>
    </div>
</div>
```

**Code-behind for menu visibility:**
```csharp
// BEFORE
foreach (RadPanelItem item in mainMenu.GetAllItems())
{
    if (item.Text == "Applicants" && user.HasRole("Applicants"))
        item.Visible = true;
}

// AFTER
navApplicants.Visible = user.HasRole("Applicants");
// Each menu item is now a server-side HtmlAnchor with runat="server"
```

---

### RadNavigation → Bootstrap navbar
```aspx
<!-- BEFORE -->
<telerik:RadNavigation ID="mainNavigation" runat="server" CssClass="header-info">
    <Nodes>
        <telerik:NavigationNode ID="nnUserName" runat="server" CssClass="user"
            SpriteCssClass="icon icon-Arrow-South" />
    </Nodes>
</telerik:RadNavigation>

<!-- AFTER -->
<nav class="navbar header-info">
    <span id="nnUserName" runat="server" class="user navbar-text" role="button"
        style="cursor:pointer;">
        <i class="bi bi-chevron-down"></i>
    </span>
</nav>
```

---

### RadTabStrip + RadMultiPage → Bootstrap tabs
```aspx
<!-- BEFORE -->
<telerik:RadTabStrip ID="RadTabStrip1" runat="server" MultiPageID="RadMultiPage1"
    SelectedIndex="0" Skin="MetroTouch">
    <Tabs>
        <telerik:RadTab Text="Profile" />
        <telerik:RadTab Text="Family" />
    </Tabs>
</telerik:RadTabStrip>
<telerik:RadMultiPage ID="RadMultiPage1" runat="server" SelectedIndex="0">
    <telerik:RadPageView ID="rpvProfile" runat="server">
        <!-- content -->
    </telerik:RadPageView>
    <telerik:RadPageView ID="rpvFamily" runat="server">
        <!-- content -->
    </telerik:RadPageView>
</telerik:RadMultiPage>

<!-- AFTER -->
<ul class="nav nav-tabs" role="tablist">
    <li class="nav-item">
        <button class="nav-link active" data-bs-toggle="tab" data-bs-target="#rpvProfile">Profile</button>
    </li>
    <li class="nav-item">
        <button class="nav-link" data-bs-toggle="tab" data-bs-target="#rpvFamily">Family</button>
    </li>
</ul>
<div class="tab-content">
    <div class="tab-pane fade show active" id="rpvProfile">
        <!-- content -->
    </div>
    <div class="tab-pane fade" id="rpvFamily">
        <!-- content -->
    </div>
</div>
```

---

### RadSplitter + RadPane → Bootstrap grid/flexbox
```aspx
<!-- BEFORE -->
<telerik:RadSplitter ID="RadSplitter2" runat="server" Width="100%" Height="100%">
    <telerik:RadPane ID="LeftPane" runat="server" Width="350">
        <!-- left content -->
    </telerik:RadPane>
    <telerik:RadSplitBar ID="RadSplitBar1" runat="server" CollapseMode="Both" />
    <telerik:RadPane ID="RadPane3" runat="server" Width="100%">
        <!-- right content -->
    </telerik:RadPane>
</telerik:RadSplitter>

<!-- AFTER -->
<div class="row h-100">
    <div class="col-md-4 border-end" id="LeftPane" runat="server">
        <!-- left content -->
    </div>
    <div class="col-md-8" id="RightPane" runat="server">
        <!-- right content -->
    </div>
</div>
```

---

### RadWindow → Bootstrap modal
```aspx
<!-- BEFORE -->
<telerik:RadWindowManager ID="RadWindowManager1" runat="server">
    <Windows>
        <telerik:RadWindow ID="MyModal" runat="server" Modal="true"
            Height="500px" Width="700px" Title="Details"
            Behaviors="Move, Resize, Close">
            <ContentTemplate>
                <!-- content -->
            </ContentTemplate>
        </telerik:RadWindow>
    </Windows>
</telerik:RadWindowManager>

<!-- AFTER -->
<div class="modal fade" id="MyModal" tabindex="-1">
    <div class="modal-dialog modal-lg">
        <div class="modal-content">
            <div class="modal-header">
                <h5 class="modal-title">Details</h5>
                <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
            </div>
            <div class="modal-body">
                <!-- content -->
            </div>
        </div>
    </div>
</div>
```

**JavaScript:** `var modal = new bootstrap.Modal(document.getElementById('MyModal')); modal.show();`

---

### RadNotification → Bootstrap toast
```aspx
<!-- BEFORE -->
<telerik:RadNotification ID="radNotification" runat="server" RenderMode="Lightweight"
    ShowCloseButton="true" AutoCloseDelay="2000" Width="350" Height="110"
    Title="Success" EnableRoundedCorners="true" Animation="Fade"
    EnableShadow="true" VisibleOnPageLoad="false" />

<!-- AFTER -->
<div class="toast-container position-fixed bottom-0 end-0 p-3">
    <div id="toastNotification" class="toast" role="alert">
        <div class="toast-header">
            <strong class="me-auto" id="toastTitle" runat="server">Success</strong>
            <button type="button" class="btn-close" data-bs-dismiss="toast"></button>
        </div>
        <div class="toast-body" id="toastBody" runat="server"></div>
    </div>
</div>
<asp:HiddenField ID="hidShowToast" runat="server" Value="" />
```

**Code-behind helper:**
```csharp
// BEFORE
radNotification.Text = "Saved!";
radNotification.Title = "Success";
radNotification.Show();

// AFTER
toastBody.InnerText = "Saved!";
toastTitle.InnerText = "Success";
hidShowToast.Value = "1";
// In Page_PreRender or via RegisterStartupScript:
ScriptManager.RegisterStartupScript(this, GetType(), "toast",
    "if(document.getElementById('" + hidShowToast.ClientID + "').value==='1'){" +
    "new bootstrap.Toast(document.getElementById('toastNotification')).show();" +
    "document.getElementById('" + hidShowToast.ClientID + "').value='';}", true);
```

---

### RadToolTip → Bootstrap tooltip/popover
```aspx
<!-- BEFORE -->
<telerik:RadToolTip ID="UserProfile" runat="server" EnableShadow="true"
    HideEvent="FromCode" IsClientID="true">
    <div class="content">
        <telerik:RadLabel ID="labFullName" runat="server" Font-Bold="true" />
    </div>
</telerik:RadToolTip>

<!-- AFTER -->
<div class="popover-body shadow p-3 d-none" id="UserProfile">
    <asp:Label ID="labFullName" runat="server" CssClass="fw-bold" /><hr />
    <!-- rest of content -->
</div>
```

---

### RadEditor → Textarea or TinyMCE/Quill
```aspx
<!-- BEFORE -->
<telerik:RadEditor ID="radEditor" runat="server" Width="100%" Height="500">
    <Tools>
        <telerik:EditorToolGroup>
            <telerik:EditorTool Name="Bold" />
            <telerik:EditorTool Name="Italic" />
        </telerik:EditorToolGroup>
    </Tools>
</telerik:RadEditor>

<!-- AFTER (using TinyMCE - free) -->
<asp:TextBox ID="radEditor" runat="server" TextMode="MultiLine" CssClass="form-control tinymce-editor"
    Width="100%" Height="500" />
<script src="https://cdn.tiny.cloud/1/no-api-key/tinymce/6/tinymce.min.js"></script>
<script>
tinymce.init({ selector: '.tinymce-editor', height: 500,
    plugins: 'lists link searchreplace',
    toolbar: 'undo redo | bold italic underline | alignleft aligncenter alignright | bullist numlist'
});
</script>
```

**Code-behind:** Access content via `radEditor.Text` (same for TextBox).

---

### RadAsyncUpload → FileUpload + Bootstrap
```aspx
<!-- BEFORE -->
<telerik:RadAsyncUpload ID="radUpload" runat="server" />

<!-- AFTER -->
<asp:FileUpload ID="radUpload" runat="server" CssClass="form-control" />
```

**Code-behind:**
```csharp
// BEFORE
if (radUpload.UploadedFiles.Count > 0) {
    UploadedFile file = radUpload.UploadedFiles[0];
    byte[] data = new byte[file.ContentLength];
    file.InputStream.Read(data, 0, data.Length);
}

// AFTER
if (radUpload.HasFile) {
    byte[] data = radUpload.FileBytes;
    string name = radUpload.FileName;
}
```

---

### RadAjaxManager + RadAjaxLoadingPanel → UpdatePanel + UpdateProgress
```aspx
<!-- BEFORE -->
<telerik:RadAjaxLoadingPanel ID="RadAjaxLoadingPanel1" runat="server" />
<telerik:RadAjaxManager ID="RadAjaxManager1" runat="server"
    OnAjaxRequest="RadAjaxManager1_AjaxRequest"
    DefaultLoadingPanelID="RadAjaxLoadingPanel1">
    <AjaxSettings>
        <telerik:AjaxSetting AjaxControlID="RadAjaxManager1">
            <UpdatedControls>
                <telerik:AjaxUpdatedControl ControlID="pnlMain" />
            </UpdatedControls>
        </telerik:AjaxSetting>
    </AjaxSettings>
</telerik:RadAjaxManager>

<!-- AFTER -->
<asp:UpdatePanel ID="upMain" runat="server">
    <ContentTemplate>
        <asp:Panel ID="pnlMain" runat="server">
            <!-- content -->
        </asp:Panel>
    </ContentTemplate>
</asp:UpdatePanel>
<asp:UpdateProgress ID="upProgress" runat="server" AssociatedUpdatePanelID="upMain">
    <ProgressTemplate>
        <div class="spinner-border text-primary" role="status">
            <span class="visually-hidden">Loading...</span>
        </div>
    </ProgressTemplate>
</asp:UpdateProgress>
```

---

### RadCodeBlock / RadScriptBlock → standard script tags
```aspx
<!-- BEFORE -->
<telerik:RadCodeBlock ID="RadCodeBlock1" runat="server">
    <script type="text/javascript">
        // code using $telerik, $find, etc.
        var grid = $find("<%= RadGrid2.ClientID %>");
    </script>
</telerik:RadCodeBlock>

<!-- AFTER -->
<script type="text/javascript">
    // Replace $find with getElementById or jQuery
    var grid = document.getElementById("<%= gvMain.ClientID %>");
</script>
```

---

### RadStepper → Bootstrap stepper (custom)
```aspx
<!-- BEFORE -->
<telerik:RadStepper ID="rsProgress" runat="server" />

<!-- AFTER -->
<div class="d-flex justify-content-between mb-4" id="rsProgress" runat="server">
    <div class="step"><span class="badge rounded-pill bg-primary">1</span> Step 1</div>
    <div class="step"><span class="badge rounded-pill bg-secondary">2</span> Step 2</div>
</div>
```

---

### RadPdfViewer → iframe or PDF.js
```aspx
<!-- BEFORE -->
<telerik:RadPdfViewer ID="RadPdfViewer1" runat="server" Width="100%" Height="600" />

<!-- AFTER -->
<iframe id="pdfViewer" runat="server" width="100%" height="600" style="border:none;"></iframe>
```

---

### RadSignature → Canvas signature (signature_pad.js)
```aspx
<!-- BEFORE -->
<telerik:RadSignature ID="RadSignature1" runat="server" />

<!-- AFTER -->
<canvas id="signaturePad" width="400" height="200" style="border:1px solid #ccc;"></canvas>
<asp:HiddenField ID="hidSignatureData" runat="server" />
<script src="https://cdn.jsdelivr.net/npm/signature_pad@4.1.7/dist/signature_pad.umd.min.js"></script>
<script>
var pad = new SignaturePad(document.getElementById('signaturePad'));
function saveSignature() {
    document.getElementById('<%= hidSignatureData.ClientID %>').value = pad.toDataURL();
}
</script>
```

---

### RadClientExportManager → html2canvas + jsPDF
```aspx
<!-- BEFORE -->
<telerik:RadClientExportManager ID="RadClientExportManager1" runat="server" />

<!-- AFTER -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
```

---

### RadRadioButtonList → RadioButtonList + Bootstrap
```aspx
<!-- BEFORE -->
<telerik:RadRadioButtonList ID="rblPayment" runat="server" />

<!-- AFTER -->
<asp:RadioButtonList ID="rblPayment" runat="server" CssClass="list-group" RepeatLayout="UnorderedList" />
```

---

## Using Directives Replacement

Remove all Telerik using statements from .cs files:
```csharp
// REMOVE these:
using Telerik.Web.UI;
using Telerik.Web.UI.Chat;
using Telerik.Web.UI.Skins;
using Telerik.Web.UI.Widgets;
using Telerik.Web.UI.Editor;
using Telerik.Web.UI.Editor.DialogControls;
using Telerik.Web.Device.Detection;
using Telerik.Web.Spreadsheet;
using Telerik.Documents.Media;
using Telerik.Windows.Documents.Spreadsheet.Expressions.Functions;
using Telerik.Windows.Documents.Flow.Model;
using Telerik.Windows.Documents.Flow.FormatProviders.Docx;
using Telerik.Windows.Documents.Flow.FormatProviders.Pdf;
using RadTreeView = Telerik.Web.UI.RadTreeView;
using RadGrid = Telerik.Web.UI.RadGrid;
using RadMenu = Telerik.Web.UI.RadMenu;
using RadPanelBar = Telerik.Web.UI.RadPanelBar;

// KEEP standard:
using System.Web.UI;
using System.Web.UI.WebControls;
```

## web.config Cleanup

Remove from `web.config`:
```xml
<!-- REMOVE from appSettings -->
<add key="Telerik.Skin" value="MetroTouch" />
<add key="Telerik.ScriptManager.TelerikCdn" value="Disabled" />
<add key="Telerik.StyleSheetManager.TelerikCdn" value="Disabled" />
<add key="Telerik.Web.UI.RenderMode" value="lightweight" />

<!-- REMOVE from pages/controls -->
<add tagPrefix="telerik" namespace="Telerik.Web.UI" assembly="Telerik.Web.UI" />

<!-- REMOVE all Telerik httpHandlers and handlers -->

<!-- ADD Bootstrap CDN if not using local -->
```

## kagorg.csproj Cleanup

Remove all Telerik `<Reference>` entries from the project file. Keep standard .NET and NuGet references.

## CSS Additions

Add to `common.css` or a new `bootstrap-overrides.css`:
```css
/* Grid row click selection */
.table-hover tbody tr { cursor: pointer; }
.table-hover tbody tr.table-primary { background-color: #cfe2ff !important; }

/* Sidebar accordion */
.accordion-button:not(.collapsed) { background-color: #212529; color: white; }
.list-group-item-action:hover { background-color: #e9ecef; }
.list-group-item img { margin-right: 8px; }

/* Toast positioning */
.toast-container { z-index: 9999; }
```

## Migration Order (recommended)

1. **MasterPage.Master** — layout, nav, scripts (affects all pages)
2. **default.aspx** — login page (simplest, good test)
3. **register.aspx** — registration form
4. **Simple pages** — forgotpassword, resetpassword, changepassword, verifyuser
5. **List pages** — applicants, employers, agents, attorneys (RadGrid-heavy)
6. **Detail pages** — applicant, cases, campaigns, applications
7. **Controls** — one at a time, testing after each
8. **Complex pages** — payments, commissions, workflows, settings

## File Checklist Per Migration

For each file being migrated:
- [ ] Replace `telerik:` markup tags with ASP.NET + Bootstrap equivalents
- [ ] Update code-behind (.cs): remove Telerik usings, change type casts, update event handlers
- [ ] Regenerate designer.cs: delete file, re-open .aspx in VS designer view, or manually update type declarations
- [ ] Remove Telerik tag prefix registrations from page directives
- [ ] Test compilation
- [ ] Test page functionality
