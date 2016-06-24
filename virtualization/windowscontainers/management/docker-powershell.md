### PowerShell para Docker

En las conversaciones que hemos mantenido con los usuarios, en los comentarios publicados en los foros, en Twitter y en GitHub, e incluso en los contactos cara a cara que hemos mantenido, hay una pregunta que se ha repetido más que ninguna otra: ¿por qué no se pueden ver los contenedores de Docker en PowerShell?

Tras debatir las ventajas, las desventajas y las diversas opciones con los usuarios, hemos llegado a la conclusión de que el módulo de contenedores de PowerShell necesita una actualización. Por esta razón, estamos dejando de usar el módulo de contenedores de PowerShell que se incluía en las versiones preliminares de Windows Server 2016 y hemos empezado a reemplazarlo por un nuevo módulo de PowerShell para Docker. El desarrollo de este nuevo módulo ya está en curso, aunque con un enfoque diferente al que adoptábamos en el pasado, y estamos llevando a cabo el trabajo de manera pública. El objetivo de este módulo es que sea el fruto de una colaboración con la comunidad y que genere una experiencia inmejorable de PowerShell para contenedores a través del motor de Docker. Este nuevo módulo se basa directamente en la interfaz de REST del motor de Docker, que permite al usuario elegir entre la CLI de Docker, PowerShell o ambos.

La creación de un módulo de PowerShell excelente no es tarea fácil, ya que es crucial generar correctamente el código y conseguir un equilibrio adecuado entre los conjuntos de parámetros y objetos y los nombres de los cmdlets. Por eso, durante el proceso de creación de este nuevo módulo, recurrimos a los usuarios finales y a las extensas comunidades de PowerShell y Docker para que nos ayuden a darle forma. ¿Qué conjuntos de parámetros son importantes para usted? ¿Debería haber un equivalente a "docker run" o se debería canalizar new-container a start-container? ¿Cuáles son sus preferencias? Para obtener más información sobre este módulo y colaborar en el desarrollo, visite nuestra página de GitHub (https://github.com/Microsoft/Docker-PowerShell/) y participe en el proceso.

A medida que progrese el desarrollo y que obtengamos un módulo sólido de primera calidad, lo publicaremos en la Galería de PowerShell y actualizaremos esta página con instrucciones de uso.






<!--HONumber=Apr16_HO4-->


