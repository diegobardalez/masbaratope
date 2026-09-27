# recetas — MasBaratoPe

Este repositorio **solo contiene las recetas** que la app MasBaratoPe usa para leer
los catálogos de las tiendas. Nada de código de la aplicación.

La app las descarga al arrancar y las **verifica con una firma Ed25519** antes de
usarlas. Si alguien modifica estos ficheros, la firma deja de cuadrar y la app
ignora lo descargado y usa las recetas que lleva dentro.

## Ficheros

| Fichero | Qué es |
|---|---|
| `recetas.json` | Los datos: qué pedir a cada tienda y cómo leer la respuesta |
| `recetas.json.sig` | Firma Ed25519 de `recetas.json` (se verifica con la clave pública) |
| `clave-publica.txt` | La clave **pública**. No es secreta: sirve para verificar, no para firmar |

## Por qué se firma y no se cifra

Cifrar no serviría: la app tendría que llevar la clave para descifrar, así que
cualquiera con el APK tendría la clave. **Firmar sí sirve**, porque la app solo
lleva la clave **pública**, que verifica pero no puede falsificar.

## Cómo se actualizan

Desde el repositorio del proyecto (no desde aquí):

```bash
# 1. editar la receta
nano public/engine/recetas.json

# 2. copiarla a este repositorio y firmarla con la clave privada
cp public/engine/recetas.json recetas/recetas.json
node tools/firmar-recetas.mjs

# 3. publicar
git add recetas/ && git commit -m "recetas: <que ha cambiado>" && git push
```

En cuanto se sube, **todas las apps cogen la receta nueva sin actualizar la APK**.

⚠️ La clave privada no está aquí ni debe estar nunca: vive en
`data/recetas-clave-privada.pem` del repositorio del proyecto. Si se pierde, hay
que generar otra y **recompilar la APK** con la pública nueva.

## Formato

Las recetas son **datos, no código**. El motor solo entiende cuatro tipos de
plataforma, así que añadir una tienda de una plataforma ya soportada es escribir
un bloque:

| `tipo` | Plataformas | Cómo se leen |
|---|---|---|
| `vtex` | Metro, Plaza Vea, Coolbox, Oechsle | API pública de catálogo (JSON) |
| `fcom` | Tottus, Falabella | `__NEXT_DATA__` del HTML |
| `next-data` | Ripley | `__NEXT_DATA__` con mapeo de campos |
| `html-magento` | Tiendas EFE | HTML de resultados con selectores |

Cada tienda declara su `base`, su `ruta` o `endpoint`, y el mapeo de campos. Al ser
datos, una receta manipulada puede devolver datos equivocados, pero **nunca
ejecutar código**.
