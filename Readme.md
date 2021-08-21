# Configuración de React Native para que funcione en procesadores M1 de Apple

1. Instalar Xcode, puede ser via AppStore o desde la página de desarrolladores de Apple.
2. Instalar las Command Line Tools, ya sea desde la página de desarrolladores de Apple o ejecutando el comando `xcode-select --install` en tu terminal.
3. Abrir xcode, ir a `Preferences`, y en la pestaña `Locations` podremos seleccionar las Command Line Tools que acabamos de instalar en el ultimo dropdown.
4. Con Hombrew, instalar una versión actualizada de Ruby: `brew install ruby` ya que la versión que trae por defecto MacOS no suele ser una versión reciente. (Sino tienes Homebrew en tu Mac, tendras que instalarlo siguiendo las instrucciones en su página web oficial)
5. Una vez terminada la instalación de Ruby con Homebrew procederemos a configurar el PATH para que tome la versión de Ruby que acabamos de instalar, abriremos nuestro archivo `.zshrc` con un editor de texto y agregaremos lo siguiente al final del mismo:
   ```bash
   export PATH=/opt/homebrew/opt/ruby/bin:/opt/homebrew/lib/ruby/gems/3.0.0/bin:$PATH
   export LDFLAGS="-L/opt/homebrew/opt/ruby/lib"
   export CPPFLAGS="-I/opt/homebrew/opt/ruby/include"
   ```
6. Ahora que tenemos Homebrew y una versión reciente de Ruby, vamos a instalar las dependencias necesarias para React Native.
   ```bash
   brew install node # Recomiendo instalar Node en su version LTS con NVM
   brew install watchman
   gem install ffi
   gem install cocoapods
   ```
7. Luego de haber instalado todo lo necesario en nuestro equipo, crearemos el proyecto de React Native.
   ```bash
   npx react-native init ProjectName
   ```
8. Lo que haremos ahora sera modificar nuestro `Podfile` en la carpeta `ios` de nuestro proyecto.

   ```Ruby
     # Enables Flipper.
     #
     # Note that if you have use_frameworks! enabled, Flipper will not work and
     # you should disable the next line.
     # Con esta nueva linea, le decimos a xcode que use una version compatible con M1
     use_flipper!({ 'Flipper' => '0.91.1', 'Flipper-Folly' => '~> 2.6', 'Flipper-RSocket' => '~> 1.4' })

     post_install do |installer|
       # Instalación para la configuración agregada
       installer.pods_project.targets.each do |target|
         target.build_configurations.each do |config|
           config.build_settings["ONLY_ACTIVE_ARCH"] = "NO"
         end
       end
       flipper_post_install(installer)
       # Aqui termina el codigo agregado
       react_native_post_install(installer)
     end
   end
   ```

9. Con esta configuración agregada a nuestro `Podfile` ahora podremos ejecutar este comando `cd ios && pod install` en nuestra consola para instalar los Pods que serviran en M1.
10. Abrimos el archivo `ProjectName.workspace` que hay dentro de la carpeta `ios` de nuestro proyecto, seleccionamos proyecto en el panel de archivos de la izquierda de `Xcode`, y damos click derecho a la carpeta con el nombre de nuestro proyecto y seleccionamos `New file`.
11. En el menu que se abrirá, seleccionamos `Swift file`. Y en el siguiente menu, verificamos que la carpeta donde se creara sea la siguiente: `ios/ProjectName/`.
12. Una vez verificado el directorio donde se creara este nuevo archivo, lo nombraremos `ForceBridgingHeader.swift`. Presionamos el boton `Create` y en el popup que saldrá daremos click al botón `Create Bridging Header`.
13. Seleccionaremos en el panel de la Izquierda de Xcode nuestro proyecto (Sera lo primero que aparecera en ese explorador de archivos) y a la derecha seleccionaremos la pestaña `Build Settings`
14. Dentro de `Build Settings` seleccionamos las pestañas `All` y `Combined`.
15. Buscamos la seccion de `Architectures` y en el apartado `Excluded Architectures` añadiremos a la key `Any iOS Simulator SDK` en `Debug` y en `Release` el valor `arm64`.
16. Finalmente, podremos cerrar Xcode y ahora, ejecutar nuestro proyecto con el comando `npx react-native run-ios`.

De esta manera podremos configurar cualquier proyecto de React Native para que sea ejecutable en nuestros entornos de desarrollo desde equipos con procesadores M1 de Apple.

[Creditos](https://medium.com/@davidjasonharding/developing-a-react-native-app-on-an-m1-mac-without-rosetta-29fcc7314d70)
