# ETL con SQL Integration Services

## Documentación de `datamartventas`

#### Creación de la Base de Datos

```sql
    CREATE DATABASE datamartventas;
    USE datamartventas;
```

## Tablas en datamartventas

### Dimensión de Productos: DimProducts

Esta tabla guarda los detalles de los productos incluyendo el ID del producto, nombre, categoría y unidades en stock.

```sql
    DROP TABLE IF EXISTS DimProducts;
    CREATE TABLE [dbo].[DimProducts](
    skProductoId INT NOT NULL IDENTITY(1,1) PRIMARY KEY,
    ProductID INT NOT NULL,
    ProductName NVARCHAR(40) NULL,
    CategoryName NVARCHAR(15) NOT NULL,
    UnitinStock SMALLINT NULL
)
```

### Dimensión de Empleados: DimEmployees

Almacena información sobre los empleados, incluidos ID, nombre completo y detalles de contacto. 
```sql
   CREATE TABLE [dbo].[DimEmployees](
    SkEmployeeId INT NOT NULL IDENTITY(1,1) PRIMARY KEY,
    EmployeeID INT NOT NULL,
    FullName NVARCHAR(50) NOT NULL,
    Address NVARCHAR(60) NULL,
    City NVARCHAR(15) NULL,
    Region NVARCHAR(15) NULL,
    Country NVARCHAR(15) NULL
)
```

### Dimensión de Clientes: DimCliente

Contiene datos sobre los clientes como ID, nombre de la compañía y detalles de contacto.
```sql
    CREATE TABLE [dbo].[DimCliente](
    SkCustomerId INT NOT NULL IDENTITY(1,1) PRIMARY KEY,
    CustomerID NCHAR(5) NOT NULL,
    CompanyName NVARCHAR(40) NOT NULL,
    Address NVARCHAR(60) NULL,
    City NVARCHAR(15) NULL,
    Region NVARCHAR(15) NULL,
    Country NVARCHAR(15) NULL
)
```

### Dimensión de Proveedores: DimProveedor

Guarda información sobre los proveedores, incluyendo ID y detalles de contacto.
```sql
    CREATE TABLE [dbo].[DimProveedor](
    SkSupplierId INT NOT NULL IDENTITY(1,1) PRIMARY KEY,
    SupplierID INT NOT NULL,
    CompanyName NVARCHAR(40) NOT NULL,
    Address NVARCHAR(60) NULL,
    City NVARCHAR(15) NULL,
    Region NVARCHAR(15) NULL,
    Country NVARCHAR(15) NULL
)
```

### Dimensión de Tiempo: DimTiempo

Mantiene registros de fechas significativas para análisis de tiempo como años, trimestres y meses.
```sql
    CREATE TABLE [dbo].[DimTiempo](
    SkTimeId INT NOT NULL IDENTITY(1,1) PRIMARY KEY,
    TimeFecha DATE NOT NULL,
    TimeAño INT NOT NULL,
    TimeTrimestre INT NOT NULL,
    TimeMes INT NOT NULL,
    TimeDescripcionMes NVARCHAR(20) NOT NULL,
    TimeDescripcionTrimestre NVARCHAR(20) NOT NULL
)
```
### Fact Table: factVentas

Tabla de hechos para ventas que enlaza todas las dimensiones anteriores con claves foráneas y almacena el precio de venta, la cantidad de unidades vendidas y el importe total.
```sql
CREATE TABLE factVentas(
    sk_Cliente INT NOT NULL,
    sk_Proveedor INT NOT NULL,
    sk_Producto INT NOT NULL,
    sk_Empleado INT NOT NULL,
    sk_timeid INT NOT NULL,
    precioVenta MONEY NOT NULL,
    cantidadUnidades INT NOT NULL,
    importe MONEY NOT NULL,
    CONSTRAINT pk_fact_ventas PRIMARY KEY(sk_Cliente, sk_Proveedor, sk_Producto, sk_Empleado, sk_TimeId),
    CONSTRAINT fk_fact_cliente FOREIGN KEY (sk_Cliente) REFERENCES DimCliente,
    CONSTRAINT fk_fact_proveedor FOREIGN KEY (sk_Proveedor) REFERENCES DimProveedor,
    CONSTRAINT fk_fact_producto FOREIGN KEY (sk_Producto) REFERENCES DimProducts,
    CONSTRAINT fk_fact_empleado FOREIGN KEY (sk_Empleado) REFERENCES DimEmployees,
    CONSTRAINT fk_fact_timeid FOREIGN KEY (sk_TimeId) REFERENCES DimTiempo
)
```
### Inserciones Iniciales

Ejemplos de cómo llenar las tablas con datos de otro sistema, en este caso, Northwind.
```sql
INSERT INTO datamartventas.dbo.DimProducts
SELECT ProductID, ProductName, UnitsInStock, CategoryName
FROM Northwind.dbo.Products AS p
INNER JOIN (SELECT CategoryID, CategoryName FROM Northwind.dbo.Categories) AS c ON p.CategoryID = c.CategoryID;
```

### Mantenimiento de la Base de Datos

Procedimientos para el borrado y reinicio de tablas en la base de datos.
```sql
TRUNCATE TABLE DimProducts;
TRUNCATE TABLE DimCliente;
TRUNCATE TABLE DimEmployees;
TRUNCATE TABLE DimProveedor;
TRUNCATE TABLE DimTiempo;
```

### Manejo de Identidades

Resetear los valores de identidad de las tablas principales después de un reinicio.
```sql
DBCC CHECKIDENT ('DimCliente', RESEED, 0);
```

### solo te deja hacer truncate  en la tabla si eliminas por lo de las foreign key
```sql
drop table factVentas
```

### borrar todos los datos
```sql
truncate table DimProducts
truncate table DimCliente
truncate table DimEmployees
truncate table DimProveedor
truncate table DimTiempo
```

### vuelves a crear la tabla factVentas
verificar que este vacios
```sql
select * from factVentas
select * from DimCliente
select * from DimProducts
select * from DimEmployees
select * from DimTiempo
```

### resetear el identity de las primary key
```sql
DBCC CHECKIDENT ('DimCliente',RESEED,0)
DBCC CHECKIDENT ('DimProducts',RESEED,0)
DBCC CHECKIDENT ('DimEmployees',RESEED,0)
DBCC CHECKIDENT ('DimProveedor',RESEED,0)
DBCC CHECKIDENT ('DimTiempo',RESEED,0)
```

### Comandos para insertar desde sql lo que se hizo en el etl 
```sql
insert into datamartventas.dbo.DimProducts
select p.productId, p.productName,p.UnitsInStock,c.CategoryName
from Northwind.dbo.Products as p
inner join (select categoryId,categoryname from northwind.dbo.Categories) as c
on p.CategoryID=c.CategoryID

insert into datamartventas.dbo.DimCliente
select CustomerID,CompanyName,Address,City,case when Region is null then 'sin región' else Region end as Region,Country from Northwind.dbo.Customers

insert into datamartventas.dbo.DimEmployees
select EmployeeID,concat(FirstName,LastName) as FullName,[Address],City,Region,Country from Northwind.dbo.Employees

insert into datamartventas.dbo.DimProveedor
select SupplierID,CompanyName,[Address],City,Region,Country from Northwind.dbo.Suppliers

insert into datamartventas.dbo.DimTiempo (TimeFecha,TimeAño,TimeTrimestre,TimeMes,TimeDescripcionMes,TimeDescripcionTrimestre) 
values (getdate(), year(getdate()),DATEPART(quarter,getdate()),month(getdate()),'mes','trimestre'); 

insert into datamartventas.dbo.factVentas	
select dc.SkCustomerId, dpv.SupplierID,de.EmployeeId,dpr.ProductID,dt.SkTimeId,od.UnitPrice,od.Quantity,od.UnitPrice * od.Quantity as Importe
from Northwind.dbo.Suppliers as dpv 
	inner join Northwind.dbo.Products as dpr
on dpv.SupplierID =dpr.SupplierID
	inner join Northwind.dbo.[Order Details] as od
on od.ProductID=dpr.ProductID
	inner join Northwind.dbo.Orders as o
on o.OrderID=od.OrderID 
	inner join Northwind.dbo.Customers as c
on c.CustomerID=o.CustomerID
	inner join datamartventas.dbo.DimCliente as dc
on dc.CustomerID=c.CustomerID
	inner join Northwind.dbo.Employees as de
on de.EmployeeID=o.EmployeeID
	inner join datamartventas.dbo.DimTiempo as dt
on dt.TimeFecha=o.OrderDate
```

