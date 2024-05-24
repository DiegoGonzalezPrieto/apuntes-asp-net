


## Interacción con Bases de Datos


### Seteos varios del proyecto

- Paginador en GridView:
	- AllowPaging="true" en GridView
	- PageSize="5"
	- OnPageIndexChanging=(evento)
		```C#
			evento_code_behind(GridViewPageEventArgs e) { 
			dgvMiView.PageIndex = e.NewPageIndex;
			dgvMiView.DataBind();
			}
		```

- Para **selección** de elemento de lista:
	- Agrega DataKeyName="Id" en en GridView
	- Agrega CommandField en las columnas con prop ShowSelectedButton="true"
	- Agrega evento OnSelectedIndexChanged en GridView
		- En code behind de este método captura el valor con:
			- dgvMiView.SelectedDataKey.Value.ToString() -> me da el Id del item

### DropDownList y Update Panel

- UpdatePanel para actualizar partes de la página en vez de toda la página.

- DDL puede ser estático o desde BD
	- Estático: asp:DropDownList que contiene varios asp:ListItem

	- Desde BD: se le sacan los asp:ListItem
		- en code behind: 
			``` C#
				page_load() { 
				ddlMiDropDown.DataSource = negocio.ListarItems(); 
				ddlMiDropDown.DataBind();
				}
			```



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


