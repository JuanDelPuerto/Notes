Búsqueda recursiva sobre nombres de directorios.
`dir -recurse | Select-Object FullName`
Seleccionar resultados de la búsqueda recursiva (como grep)
`dir -recurse | Select-Object FullName | Select-String "Sticky"`