

# Introduction to Linux

## Preparation

1. Boot from a usb stick (or live cd), we suggest to use  [Ubuntu gnome](http://ubuntugnome.org/) distribution, or another ubuntu derivative.

2. (Optional) Configure keyboard layout and software repository
   Go to the the *Activities* menu (top left corner, or *start* key):
      -  Go to settings, then keyboard. Set the layout for latin america
      -  Go to software and updates, and select the server for Colombia
3. (Optional) Instead of booting from a live Cd. Create a partition in your pc's hard drive and install the linux distribution of your choice, the installed Os should perform better than the live cd.

## Introduction to Linux

1. Linux Distributions

   Linux is free software, it allows to do all sort of things with it. The main component in linux is the kernel, which is the part of the operating system that interfaces with the hardware. Applications run on top of it. 
   Distributions pack together the kernel with several applications in order to provide a complete operating system. There are hundreds of linux distributions available. In
   this lab we will be using Ubuntu as it is one of the largest, better supported, and user friendly distributions.


2. The graphical interface

   Most linux distributions include a graphical interface. There are several of these available for any taste.
   (http://www.howtogeek.com/163154/linux-users-have-a-choice-8-linux-desktop-environments/).
   Most activities can be accomplished from the interface, but the terminal is where the real power lies.

### Playing around with the file system and the terminal
The file system through the terminal
   Like any other component of the Os, the file system can be accessed from the command line. Here are some basic commands to navigate through the file system

   -  ``ls``: List contents of current directory
   - ``pwd``: Get the path  of current directory
   - ``cd``: Change Directory
   - ``cat``: Print contents of a file (also useful to concatenate files)
   - ``mv``: Move a file
   - ``cp``: Copy a file
   - ``rm``: Remove a file
   - ``touch``: Create a file, or update its timestamp
   - ``echo``: Print something to standard output
   - ``nano``: Handy command line file editor
   - ``find``: Find files and perform actions on it
   - ``which``: Find the location of a binary
   - ``wget``: Download a resource (identified by its url) from internet 

Some special directories are:
   - ``.`` (dot) : The current directory
   -  ``..`` (two dots) : The parent of the current directory
   -  ``/`` (slash): The root of the file system
   -  ``~`` (tilde) :  Home directory
      
Using these commands, take some time to explore the ubuntu filesystem, get to know the location of your user directory, and its default contents. 
   
To get more information about a command call it with the ``--help`` flag, or call ``man <command>`` for a more detailed description of it, for example ``man find`` or just search in google.


## Input/Output Redirections
Programs can work together in the linux environment, we just have to properly 'link' their outputs and their expected inputs. Here are some simple examples:

1. Find the ```passwd```file, and redirect its contents error log to the 'Black Hole'
   >  ``find / -name passwd  2> /dev/null``

   The `` 2>`` operator redirects the error output to ``/dev/null``. This is a special file that acts as a sink, anything sent to it will disappear. Other useful I/O redirection operations are
      -  `` > `` : Redirect standard output to a file
      -  `` | `` : Redirect standard output to standard input of another program
      -  `` 2> ``: Redirect error output to a file
      -  `` < `` : Send contents of a file to standard input
      -  `` 2>&1``: Send error output to the same place as standard output

2. To modify the content display of a file we can use the following command. It sends the content of the file to the ``tr`` command, which can be configured to format columns to tabs.

   ```bash
   cat milonga.txt | tr '\n' ' '
   ```
   
## SSH - Server Connection

1. The ssh command lets us connect to a remote machine identified by SERVER (either a name that can be resolved by the DNS, or an ip address), as the user USER (**vision** in our case). The second command allows us to copy files between systems (you will get the actual login information in class).

   ```bash
   
   #connect
   ssh USER@SERVER
   ```

2. The scp command allows us to copy files form a remote server identified by SERVER (either a name that can be resolved by the DNS, or an ip address), as the user USER. Following the SERVER information, we add ':' and write the full path of the file we want to copy, finally we add the local path where the file will be copied (remember '.' is the current directory). If we want to copy a directory we add the -r option. for example:

   ```bash
   #copy 
   scp USER@SERVER:~/data/sipi_images .
   
   scp -r USER@SERVER:/data/sipi_images .
   ```
   
   Notice how the first command will fail without the -r option

See [here](ssh.md) for different types of SSH connection with respect to your OS.

## File Ownership and permissions   

   Use ``ls -l`` to see a detailed list of files, this includes permissions and ownership
   Permissions are displayed as 9 letters, for example the following line means that the directory (we know it is a directory because of the first *d*) *images*
   belongs to user *vision* and group *vision*. Its owner can read (r), write (w) and access it (x), users in the group can only read and access the directory, while other users can't do anything. For files the x means execute. 
   ```bash
   drwxr-x--- 2 vision vision 4096 ene 25 18:45 images
   ```
   
   -  ``chmod`` change access permissions of a file (you must have write access)
   -  ``chown`` change the owner of a file
   
## Sample Exercise: Image database

1. Create a folder with your Uniandes username. (If you don't have Linux in your personal computer)

2. Copy *sipi_images* folder to your personal folder. (If you don't have Linux in your personal computer)

3.  Decompress the images (use ``tar``, check the man) inside *sipi_images* folder. 

4.  Use  ``imagemagick`` to find all *grayscale* images. We first need to install the *imagemagick* package by typing

    ```bash
    sudo apt-get install imagemagick
    ```
    
    Sudo is a special command that lets us perform the next command as the system administrator
    (super user). In general it is not recommended to work as a super user, it should only be used 
    when it is necessary. This provides additional protection for the system.
    
    ```bash
    find . -name "*.tiff" -exec identify {} \; | grep -i gray | wc -l
    ```
    
3.  Create a script to copy all *color* images to a different folder
    Lines that start with # are comments
       
      ```bash
      #!/bin/bash
      
      # go to Home directory
      cd ~ # or just cd

      # remove the folder created by a previous run from the script
      rm -rf color_images

      # create output directory
      mkdir color_images

      # find all files whose name end in .tif
      images=$(find sipi_images -name *.tiff)
      
      #iterate over them
      for im in ${images[*]}
      do
         # check if the output from identify contains the word "gray"
         identify $im | grep -q -i gray
         
         # $? gives the exit code of the last command, in this case grep, it will be zero if a match was found
         if [ $? -eq 0 ]
         then
            echo $im is gray
         else
            echo $im is color
            cp $im color_images
         fi
      done
      
      ```
      -  save it for example as ``find_color_images.sh``
      -  make executable ``chmod u+x`` (This means add Execute permission for the user)
      -  run ``./find_duplicates.sh`` (The dot is necessary to run a program in the current directory)
      

## Your turn

1. What is the ``grep``command?

El comando ``grep`` sirve para encontar en un archivo un patrón, imprimiendo todas las líneas que contengan esa  palabra. Su uso más común es para buscar información en una base de datos simple o en un archivo estructurado.

![](https://github.com/ldalfonso/IBIO4490/blob/master/01-Linux/1.PNG)

2. What is the meaning of ``#!/bin/python`` at the start of scripts?

La línea ``#!/bin/phyton`` se refiere a que el script a trabajar debe ser corrido en Phyton para obtener los resultados queridos.

3. Download using ``wget`` the [*bsds500*](https://www2.eecs.berkeley.edu/Research/Projects/CS/vision/grouping/resources.html#bsds500) image segmentation database, and decompress it using ``tar`` (keep it in you hard drive, we will come back over this data in a few weeks).

Para descargar la base de datos mencionada se utilizó el comando ``wget`` y el url de la página que permite la descarga de [*bsds500*]. Por otro lado, para descomprimir este archivo se utilizó el comando ``tar`` con su combinación ``-xf``, obteniendo una carpeta de archivos con el nombre de BSR.

![](https://github.com/ldalfonso/IBIO4490/blob/master/01-Linux/3.1.PNG)

![](https://github.com/ldalfonso/IBIO4490/blob/master/01-Linux/3.2.PNG)

4. What is the disk size of the uncompressed dataset, How many images are in the directory 'BSR/BSDS500/data/images'?

Para conocer el tamaño en el disco de la carpeta anteriormente descomprimida, BSR, se utilizó el comando ``du`` con su combinación ``-hs``, el cual se encarga de mostrar el total del espacio utilizado de una carpeta, en un formato humanamente legible [1], obteniendo un resultado de 73M. Por otro lado, para conocer el número de imágenes del directorio 'BSR/BSDS500/data/images', se utilizó el comando ``find``, el cual se encarga de buscar los archivos que tienen en su nombre .jpg, posteriormente se utiliza el comando ``identify`` para que identifique los archivos encontrados anteriormente y con el comando ``wc`` y su combinación ``-l``, se cuentan el número de archivos encontrados anteriormente, obteniendo así un resultado de 500 imágenes.

![](https://github.com/ldalfonso/IBIO4490/blob/master/01-Linux/4.PNG)

5. What are all the different resolutions? What is their format? Tip: use ``awk``, ``sort``, ``uniq``

Para encontrar las diferentes resoluciones de las imágenes se utilizaron los mismos comandos del punto anterior a excepción de ``wc`` e incluyendo ``awk``, ``sort`` y ``uniq``. El comando de ``awk`` se encarga de imprimir una columna específicas de la línea de texto obtenida con los comandos de ``find`` e ``identify`` [2]. Por otro lado, el comando de ``sort`` toma una lista de elementos y los ordena alfabéticamente y numéricamente. Por último, el comando de ``uniq`` toma una lista de elementos y elimina las líneas duplicadas. Con lo anterior se obtuvo como resultado que las resoluciones de las imágenes son de 321x481 y 481x321, mientras que el formato de las mismas es de JPEG [3].

![](https://github.com/ldalfonso/IBIO4490/blob/master/01-Linux/5.PNG)

6. How many of them are in *landscape* orientation (opposed to *portrait*)? Tip: use ``awk`` and ``cut``

Para encontrar la cantidad de imágenes que se encuentran en una orientación de landscape se creo un script con el comando ``touch``, el cual primero creaba dos carpetas que reciben como nombre "landscape" y "portrait", para posteriormente guardar las imágenes. Luego, por medio de un recorrido sobre las imágenes, las cuales fueron detectadas con el comando ``find``, se identifico si en la línea de texto existía la orientación 481x321, esto con los comandos ``identify`` y ``grep``, entonces si lo anterior ocurría la imagen era copiada a la carpeta landscape y si no era copiada a la carpeta portrait . Por último al finalizar el ciclo, por medio del comando ``wc -l`` se contaban el número de imágenes en la carpeta de landscape, lo cual dió como resultado 348.

![](https://github.com/ldalfonso/IBIO4490/blob/master/01-Linux/6.PNG)

![](https://github.com/ldalfonso/IBIO4490/blob/master/01-Linux/6.1.PNG)

7. Crop all images to make them square (256x256) and save them in a different folder. Tip: do not forget about  [imagemagick](http://www.imagemagick.org/script/index.php).

Para lograr el objetivo de este punto se creo un script con el comando ``touch``, el cual crea una carpeta llamada "crop_images", luego, dentro de esta carpeta crea una nueva llamada "images" y dentro de esta crea tres carpetas correspondientes a "test", "train" y "val". Luego de crear las carpetas mencionadas se realizo un recorrido utilizando el comando ``ls``, el cual se encarga de listar todos los archivos que están en el directorio actual y por último con el comando ``convert`` y ``crop`` se cortan las imágenes a 256x256 y se mandan a la nueva carpeta creada con el mismo nombre.

![](https://github.com/ldalfonso/IBIO4490/blob/master/01-Linux/7.PNG)

# Referencias

[1] J. B., "Los comandos SSH usados más frecuentemente y su utilización", KB.IWEB.COM, 2016. [Online]. Disponible en: https://kb.iweb.com/hc/es/articles/230241928-Los-comandos-SSH-usados-m%C3%A1s-frecuentemente-y-su-utilizaci%C3%B3n. 

[2] S. Suri, "Using awk over a remote ssh connection", A system engineer's notebook, 2017. [Online]. Disponible en: http://asystemengineersnotebook.blogspot.com/2017/01/using-awk-over-remote-ssh-connection.html. 

[3] Linode, "Manipulate Lists with sort and uniq", Linode Guides & Tutorials, 2018. [Online]. Disponible en: https://www.linode.com/docs/tools-reference/tools/manipulate-lists-with-sort-and-uniq/. 


# Report

For every question write a detailed description of all the commands/scripts you used to complete them. DO NOT use a graphical interface to complete any of the tasks. Use screenshots to support your findings if you want to. 

Feel free to search for help on the internet, but ALWAYS report any external source you used.

Notice some of the questions actually require you to connect to the course server, the login instructions and credentials will be provided on the first session. 

## Deadline

We will be delivering every lab through the [github](https://github.com) tool (Silly link isn't it?). According to our schedule we will complete that tutorial on the second week, therefore the deadline for this lab will be specially long **February 7 11:59 pm, (it is the same as the second lab)** 

### More information on

http://www.ee.surrey.ac.uk/Teaching/Unix/ 




