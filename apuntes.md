
### Seteos varios del proyecto

#### Paginador en GridView:
	- AllowPaging="true" en GridView
	- PageSize="5"
	- OnPageIndexChanging=(evento)
		```C#
			evento_code_behind(GridViewPageEventArgs e) { 
			dgvMiView.PageIndex = e.NewPageIndex;
			dgvMiView.DataBind();
			}
		```

#### Para **selección** de elemento de lista:
	- Agrega DataKeyName="Id" en en GridView
	- Agrega CommandField en las columnas con prop ShowSelectedButton="true"
	- Agrega evento OnSelectedIndexChanged en GridView
		- En code behind de este método captura el valor con:
			- dgvMiView.SelectedDataKey.Value.ToString() -> me da el Id del item

### DropDownList y Update Panel


#### DDL puede ser estático o desde BD
	- Estático: asp:DropDownList que contiene varios asp:ListItem

	- Desde BD: se le sacan los asp:ListItem
		- en code behind: 
			``` C#
				page_load() { 
				ddlMiDropDown.DataSource = negocio.ListarItems(); 
				ddlMiDropDown.DataBind();
				}
			```
#### DDL Enlazados
	- Elegis algo en primer ddl y eso cambia opciones en otro ddl
	- El primer DDL asp:DropDownList tiene el AutoPostBack="true"
	para que el DDL dispare el postback
		- y OnSelectedIndexChanged=(evento)

	- El segundo DDL está normal, pero tiene ID

	- codebehind en page_load:
	```C#
	page_load() {
		// solo si NO es postback
		ddlUno.DataSource = negocio.listar();
		ddlUno.DataTextField = "Descripcion"; // lo que muestra el DDL
		ddlUno.DataValueField = "Id"; // el valor que queda seleccionado por abajo
		ddlUno.DataBind();
	}
	```
	- codebehind en evento de OnSelectedIndexChanged:
	```C#
	OnSelectedIndexChanged...() {
		int id = int.Parse(ddlUno.SelectedItem.Value);
		//	... filtrar la lista que va en el ddlDos
		ddlDos.DataBind();
	}

	```

#### UpdatePanel para actualizar partes de la página en vez de toda la página.
	- Actualiza un fragmento del HTML solamente.

	- en .aspx necesitamos un asp:ScriptManager (nada más) para que esté el JS necesario

	- en .aspx se agrega un <asp:UpdatePanel> que contiene un <ContentTemplate>
		- dentro del ContentTemplate va todo el html que debe actualizarse
		y los postbacks que deben capturarse


Se puede combinar los DDL enlazados con UpdatePanel (dejarlos a ambos adentro
de un UpdatePanel) para que se actualice solamente esa parte de la página.

#### Seleccionar programáticamente el valor de un DDL

En code behind se puede cambiar el indice seleccionado:

```C#
ddlUno.SelectedIndex = ddlUno.Items.IndexOf(ddlUno.Items.FindByValue(id));
```

O sino:

```C#
ddlUno.SelectedIndex = -1; // borra selección
ddlUno.Items.FindByValue(id).Selected = true;
```

## Interacción con Bases de Datos

### Crear Pokemon

1. Deshabilitar campos (como id) no editables
	```
	txtId.Enabled = false;
	```
2. Cargar desplegables. (Si no es Postback)

`ddl.DataValueField = "Id"` y `ddl.DataTextField="Nombre"`

`ddl.DataBind()`

3. Evento Aceptar:

Debe capturar todos los valores del formulario en un objeto del modelo creado.
Luego con el objeto de negocio crear el nuevo registro.

#### Manejo de Error

Try Catch y además guardar el la excepción en la session.  
Luego redirigir a /error con ese dato.


### Modificar Pokemon

Seleccionar un item, pregargar los datos en un formulario, hacer el update.

El id del item puede venir en la URL del Alta, para en ese caso cargar los datos
del registro y permitir modificar.

- **Importante:** el precargado de datos a editar debe hacer solamente
si NO es Postback.

```C#
// if (!IsPostBack) ...
if (Request.QuerString["id"] != null) // ... cargar datos

// x ej: negocio.getById(int.Parse(Request.QueryString["id"]))
// txtNombre.Text = miItem.Nombre;
// ...
```

Para precargar los valores correspondientes en los dropdown, 
tienen que estar cargados con todos los valores.

Luego:
```C#
ddlTipo.SelectedIndex = ddlTipo.Items.IndexOf(ddlUno.Items.FindByValue(id));
// u otro método p/ seleccionar programáticamente un valor
```

Luego enviar los datos (id incluido) a `negocio.modificar(miRegistro)`

### Eliminar Pokemon (Baja física)

Agrega un botón de "Eliminar" en la página de Alta/Edición.

**Confirmación de eliminación** puede ser con UpdatePanel.


### Inactivar Pokemon (Baja lógica)

Se cambia la bandera de Activo en el registro.
Se muestran todos los registros pero indicando si están activos o no...
Permite una reactivación.

### Reactivar Pokemon

El texto del botón "Inactivar" debe ser "Rectivar" 
cuando el registro está inactivo. (Y viceversa)


### Filtro rápido

Que filtre automáticamente a medida que escribimos.

1. Agregar asp:TextBox con AutoPostBack="true" OnTextChanged="(evento)"
2. En el evento de OnTextChanged:
```C#
lsitaFiltrada = listaRegistros.FindAll(r => 
	r.Nombre.ToUpper().Contains(txtFiltro.Text.ToUpper()))

dgvMiView.DataSource = lsitaFiltrada;
dgvMiView.DataBind();
```
3. Como extra, podría usarse UpdatePanel para actualizar solo el DGV


### Filtro avanzado

Para mostrar u ocultar os filtros usa un CheckBox con AutoPostBack="true"
y con OnCheckedChanged=(evento). En el evento se marca una bandera
FiltroAvanzado: bool como true/false.
Y con un IF muestra y oculta los divs con el formulario.
> Como alternativa puede usarse el atributo Checked del CheckBox
> en vez de la bandera.

Luego el filtro se compone de varios DDL enlazados para definir el campo
y el criterio mediante el cual se va a filtrar.

**Atención:** el estado de la bandera FiltroAvanzado puede perderse en el
PostBack.

- Se puede agregar un botón de Limpiar Filtros

### Login y Permisos

#### Login

Luego de chequear las credenciales contra la base de datos, si el login es
exitoso, se agregan los datos del usuario a la Session.
Finalmente un redirect a la página de login exitoso.

Luego en las páginas protegidas se puede revisar la Session para ver si
el usuario está logueado y si tiene permisos para ingresar.

#### Permisos

Los permisos se manejan a nivel de página o por fragmentos de páginas
en base al tipo de usuario logueado, o a su ID.

#### Logout

Sería sacar el objeto de la Session.

### Envío de emails

Usar una clase EmailServicio:
- Tiene un atributo `SmtpClient`
- Otro atributo `MailMessage`
- Se le configura una cuenta de salida (gmail funciona):
``` C#
constructor() {
	smtpClient.Credentials = new NetworkCredential("user@....", "pass");
	smtpClient.EnableSsl = true;
	smtpClient.Port = 587;
	server.Host = "smtp.gmail.com";
}

```

- Se arma el correo a enviar:
``` C#
email = new MailMessage();
email.From = new MailAddress("user@...");
email.To.Add(destinatario);
email.Subject = "asunto";
email.IsBodyHtml = true;
email.Body = "<h1>Hola, este es un correo automático.</h1>";
```


### Registro Trainee - output SQL - MailTrap

### MailTrap Configuraciones

### Login y Acceso a Pantallas PokeApp

### AutoLogin en Registro

### Subir Imagen a Perfil

### Agregar Fecha Nacimiento a Perfil

### Enviar NULL a DB


## Validaciones


### Cómo validar en la Web [Texto]

### Manejo de Errores

### Manejo de Errores Genérico

### Niveles de Validación

### Validaciones Clásicas

### ASP Validators: Required

### Validators Range y RegEx

### Validaciones con JS + Styles


