# Apuntes de ASP.Net

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

El objeto User que esté en la Session debe tener todos los datos pertinentes
del usuario. Como:
- ID
- Tipo de usuario
- Nombre de usuario
- Mail de usuario

#### Permisos

Los permisos se manejan a nivel de página o por fragmentos de páginas
en base al tipo de usuario logueado, o a su ID.

#### Logout

Sería sacar el objeto de la Session.

### Envío de emails

> Ojo porque Gmail parece que no permite el envío como está indicado aquí!

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
email.From = new MailAddress("no-responder@..."); // puede ser nombre de fantasía
email.To.Add(destinatario);
email.Subject = "asunto";
email.IsBodyHtml = true;
email.Body = "<h1>Hola, este es un correo automático.</h1>";
```


### Registro Trainee - output SQL - MailTrap

#### Insert y Output

Debe insertar un registro en la base de datos. Para obtener el id del registro
insertado, se agrega `output inserted.id` justo antes de `VALUES` en la
sentencia de `INSERT`. Luego en el código de C# hay que ejecutar la sentencia
como `comando.ExecuteScalar()` que devuelve el valro retornado por la BD.

#### Correo con MailTrap

Para enviar un correo se usa el serivicio MailTrap.
Tiene una integración con C# que brinda el snippet de código para el EmailServicio

> Ojo porque hay una opción SandBox que es de pruebas y no hace os envíos.


### MailTrap Configuraciones

Para las pruebas se puede usar SandBox que no envía mensajes sino que muestra
únicamente si el mensaje se enviaría correctamente. No consume la cuota de 500
envíos gratuitos que brinda MailTrap.


### Login y Acceso a Pantallas PokeApp

Para el login se buscan las credenciales en la base de datos. Si es exitoso,
se obtienen todos los datos del usuario (incluído el ID) se guardan en Session.


#### Seguridad (clase estática)

Este clase puede validar si hay una sesión activa.
También puede validar si el usuario logueado tiene acceso a cada página en
particular o a ver fragmentos de la página.

##### Validar sesión a nivel general en Master Page

Se puede validar que el usuario esté logueado directamente en la Master Page.
Pero para eso **hay que exceptuar las páginas que son accesibles sin loguearse**.
```C#
// en page_load de la Master Page

// la propiedad Page contiene la página que se está por cargar

if (Page is Login) { // Page es instancia de la página Login?

	// dejar pasar aunque no esté logueado
}

```

> **El logout se hace con `Session.Clear()` para vaciar la sesión**

##### Validar si es Admin en Seguridad

Se puede usar en las páginas que son exclusivas para Admin.

##### También hay que ocultar botones de login cuando hay sesión activa



### AutoLogin en Registro


Que se loguee automáticamente luego de registrarse:

1. Luego de insertar el usuario en Registro, capturar el Id que devuelve 
el INSERT.
2. Guardar datos del usuario en Session["user"].
3. Redirigir a una pantalla ya ingresada.


### Subir Imagen a Perfil

1. Para recibir el archivo de usuario:
```ASP
<input type=file id="txtImagen" runat=server class="form-control"/>
```

2. Para mostrar la imagen
```ASP
<asp:Image ID="..." ImageUrl="placeholder.png" 
	runat="server" CssClass="image-fluid"/> 
```

3. Crear carpeta /imagenes en el proyecto webforms

4. Disparar un evento al clickear Guardar el formulario.

5. En ese evento:
	1. Recuperar el directorio base de las imagenes.
	```C#
	// devuelve ruta de carpeta de imagenes
	string rutaImagenes = Server.MapPath("./imagenes"); 
	```
	2. Guardar archivo:
	```C#
	// ver de obtener la extensión del archivo
	txtImagen.PostedFile.SaveAs(ruta + "perfil-" + user.Id + .jpg);
	```
	3. Guardar la relación entre la imagen y el usuario en la BD.

6. Cargar la imagen almacenada donde sea necesario.
```C#
// la virgulilla refiere a la raíz del proyecto webforms
imgPerfil.ImageUrl = "~/imagenes/" + "perfil-" + user.Id + .jpg
```
> La ruta de la imagen del usuario logueado se puede guardar en la Session.

#### Acceder a elementos de la Master Page desde otra paǵina

```C#
Label lbl = (Label)Master.FindControl("lblNombreUsuario");
```

### Agregar Fecha Nacimiento a Perfil

Input:
`<asp:TextBox ID="txtFecha" ... TextMode="date">`

Lectura:
`DateTime fecha = DateTime.Parse(txtFecha.Text);`

Lectura de Base de Datos:
`usuario.Fecha = DateTime.Parse(datos.Lector["fecha"].ToString());`

Precargar un TextBox de TextMode="date":
` txtFecha.Text = usuario.Fecha.ToString("yyyy-MM-dd"); // !`

### Enviar NULL a DB

Puede ser con un operador ternario:

```C#
datos.setearParametro("@nombre", 
	user.Nombre != null ? user.Nombre : (object)DBNull.Value)
```

O usando **null coalescing**:
```C#
datos.setearParametro("@nombre", 
	(object)user.Nombre ?? DBNull.Value)
```


## Validaciones

### Cómo validar en la Web [Texto]

(ver texto en campus)


### Manejo de Errores

* En la Master (donde filtramos qué páginas son accesibles para usuarios logueados):

	`if (Page is Error) { // deja pasar aunque no esté logueado }`

* Si hay excepción, guardar mensaje de error en `Session["error"]` y redirigir a Error.aspx


### Manejo de Errores Genérico

En caso de que se escape un error, se puede usar Error Handling.

> Se agrega un bloque de código predefinido en el archivo `Global.asax.cs` (ver docs)
> 
> Hay que manejar el error como se quiera.

Tamién se puede manejar a nivel de la Page, con un evento Page_Error()


### Niveles de Validación

#### En HTML o JS

Agregar `required` a un campo input. O agregar `min` / `max` en el caso de numeros.

> ! El problema es que el HTML / JS se puede manipular. Hay que hacer validación en back.

#### En CodeBehind


#### En el modelo (lanzando excepciones desde clases de dominio)


#### En Base de Datos


### Validaciones Clásicas

### ASP Validators: Required

### Validators Range y RegEx

### Validaciones con JS + Styles


