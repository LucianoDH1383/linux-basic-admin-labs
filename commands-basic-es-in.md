
# Basic File System Commands / Comandos Básicos del Sistema de Archivos

## ls – List files and directories / Listar archivos y directorios
Lists files and directories in a directory.
Lista archivos y directorios en un directorio.
```bash
ls
ls -l
ls -a
ls -la
ls -lh
ls -ltr
```
## cd – Change directory / Cambiar directorio

Changes the current working directory.
Cambia el directorio de trabajo actual.
```bash

cd /path/to/directory
cd ..
cd ~
cd -
cd
```
## pwd – Print working directory / Mostrar directorio actual

Prints the full path of the current directory.
Muestra la ruta completa del directorio actual.
```bash

pwd
pwd -P
```
## mkdir – Make directory / Crear directorios

Creates new directories.
Crea nuevos directorios.
```bash

mkdir directory_name
mkdir -p path/to/new/directory
mkdir dir1 dir2 dir3
mkdir -m 755 secure_dir
```
## rm – Remove files and directories / Eliminar archivos y directorios

Removes files and directories.
Elimina archivos y directorios.
```bash

rm file.txt
rm -r directory
rm -f file.txt
rm -rf directory
rm -i file.txt
```
## cp – Copy files and directories / Copiar archivos y directorios

Copies files and directories to another location.
Copia archivos y directorios a otra ubicación.
```bash

cp source.txt destination.txt
cp -r source_dir/ destination_dir/
cp -i source.txt destination.txt
cp -u source.txt destination.txt
```
## mv – Move or rename files / Mover o renombrar archivos

Moves or renames files and directories.
Mueve o renombra archivos y directorios.
```bash

mv old_name.txt new_name.txt
mv file.txt /destination/path/
mv -i old_name.txt new_name.txt
mv file1.txt file2.txt destination_dir/
```
## cat – Concatenate files / Concatenar archivos

Displays the contents of files.
Muestra el contenido de los archivos.
```bash

cat file.txt
cat file1.txt file2.txt
cat -n file.txt
cat > new_file.txt
```
## touch – Create empty files / Crear archivos vacíos

Creates new empty files or updates timestamps.
Crea archivos vacíos o actualiza marcas de tiempo.
```bash

touch file.txt
touch file1.txt file2.txt
touch -t 202401010000 file.txt
```
## find – Find files and directories / Buscar archivos y directorios

Searches for files and directories.
Busca archivos y directorios.
```bash

find . -name "*.txt"
find /path -type f -name "pattern"
find . -size +100M
find . -mtime -7
```
## less / more – View file content page by page / Ver contenido página por página

Displays file content one page at a time.
Muestra el contenido de archivos página por página.
```bash

less file.txt
more file.txt
```
## head – Show first lines / Mostrar primeras líneas

Displays the first lines of a file.
Muestra las primeras líneas de un archivo.
```bash

head file.txt
head -n 20 file.txt
head -c 100 file.txt
```
## tail – Show last lines / Mostrar últimas líneas

Displays the last lines of a file.
Muestra las últimas líneas de un archivo.
```bash

tail file.txt
tail -n 20 file.txt
tail -f logfile.log
```
## ln – Create links / Crear enlaces

Creates symbolic or hard links.
Crea enlaces simbólicos o duros.
```bash

ln -s /path/to/source link_name
ln source_file hard_link
ln -sf new_source existing_link
```
## file – Determine file type / Determinar tipo de archivo

Determines the type of a file.
Determina el tipo de un archivo.
```bash

file document.pdf
file script.sh
file image.jpg
```
## du – Disk usage / Uso de disco

Shows disk usage of files and directories.
Muestra el uso de disco de archivos y directorios.
```bash

du -sh directory/
du -h file.txt
du -ah /path
```
## df – Disk free space / Espacio libre en disco

Shows free disk space on filesystems.
Muestra espacio libre en disco en sistemas de archivos.
```bash

df -h
df -Th
df -i
```
##stat – File statistics / Estadísticas de archivos

Displays detailed file information.
Muestra información detallada de archivos.
```bash

stat file.txt
stat -c "%A %n" *
```
## tree – Display directory tree / Mostrar árbol de directorios

Displays directory structure as a tree.
Muestra la estructura de directorios como un árbol.
```bash

tree
tree -L 3
tree -d
```
## locate – Find files quickly / Buscar archivos rápidamente

Finds files by name quickly.
Busca archivos por nombre rápidamente.
```bash

locate filename
locate "*.conf"
locate -i document
```
## which – Locate a command / Localizar un comando

Shows the full path of shell commands.
Muestra la ruta completa de comandos de shell.
```bash

which ls
which python
which -a bash
```
## whereis – Locate binary, source, and manual / Localizar binario, fuente y manual

Locates binary, source, and manual page files.
Localiza archivos binarios, fuentes y páginas de manual.
```bash

whereis ls
whereis -b python
whereis -m bash
```
