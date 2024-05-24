




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

### Modificar Pokemon

### Eliminar Pokemon

### Inactivar Pokemon

### Reactivar Pokemon

### Filtro rápido

### Filtro avanzado

### Login y Permisos

### Envío de emails

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


