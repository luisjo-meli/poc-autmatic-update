# Prueba de Concepto Auto Close Pr Dependabot

Esta es una prueba de concepto en  trabajamos el cierre automático de pr abiertos por dependabot.



## Herramienta usada

Para esta PoC usamos el actions: [fastify/github-action-merge-dependabot@v3.2](https://github.com/fastify/github-action-merge-dependabot)
Este action creado por el equipo de Fastify esta escrito en nodejs, tiene como ventajas un bajo consumo de recursos, alta estabilidad y mantenibilidad.

## Configuraciones necesarias.

```
name: auto-merge  
on: [pull_request]  
jobs:  
  auto-merge:  
    permissions:  
      contents: write  
      pull-requests: write  
    runs-on: ubuntu-latest  
    steps:  
      - uses: fastify/github-action-merge-dependabot@v3.2  
        with:  
          github-token: ${{ secrets.GITHUB_TOKEN }}  
          target: patch
```
Es necesario proveer el github_token para que pueda tomar los permisos necesarios para cerrar el pr
## Escenarios de las pruebas y resultados.

Para esta PoC me centre en los siguientes escenarios.

- Mezclar PR cuando es abierto por dependabot, es una versión patch y no hay intervención externa.
- Mezclar PR cuando es abierto por dependabot, es una versión patch y hay intervención externa.
- Mezclar PR cuando es abierto por dependabot y es una versión diferente a la patch.


### Mezclar PR cuando es abierto por dependabot, es una versión patch y no hay intervención externa.

Configuramos dependabot para que corriera en cierto periodo de tiempo, agregamos dependencias en versiones desactualizadas, pero solo por version patch y esperamos que se creara el pr. Una vez creado el PR, se ejecuta la tarea de auto-merge, la cual se encarga de validar si es un pr por dependabot, si tiene una version patch para actualizar y si nadie además de dependabot a intervenido.

Podemos ver el resultado en este [PR](https://github.com/luisjo-meli/poc-autmatic-update/actions/runs/3231468004/jobs/5291028286) donde se aprecia que el pr fue cerrado de forma correcta.


### Mezclar PR cuando es abierto por dependabot, es una versión patch y hay intervención externa.

Para este escenario, creamos a un pr de dependabot existente y sin cerrar, le hicimos un commit de prueba y luego agregamos el auto-merge, esto con el propósito de validar si podía cerrar un pr ya abierto y con intervención de alguien diferente a dependabot.

El resultado se puede ver en este [PR](https://github.com/luisjo-meli/poc-autmatic-update/pull/10) donde podemos observar que al tener commits de distintas fuentes, el actions finaliza correctamente deja el siguiente mensaje: `Warning: PR contains non dependabot commits, skipping.` dejando asi el pr abierto.



### Mezclar PR cuando es abierto por dependabot y es una versión diferente a la patch.

Uno de los puntos a validar es como reacciona la herramienta cuando hay un PR de dependabot con una version diferente a patch, por ejemplo, con una version minor.

Para este caso usamos spring-boot-starter-mail from 2.6.12,  una vez que corrienda dependabot, este generaria un pr para actualizar spring-boot-starter-mail from a la version 2.7.4.  [PR](https://github.com/luisjo-meli/poc-autmatic-update/pull/14), para estos casos, se ejecuta el auto-merge, pero al detectar que la versión que se actualiza es diferente a patch, finaliza la tarea y deja el siguiente mensaje: `Warning: Target specified does not match to PR, skipping.` 
