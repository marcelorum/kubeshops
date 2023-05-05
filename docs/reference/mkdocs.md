# Ejemplos de mkdocs

Esta página incluye algunos buenos trucos que puedes hacer con `mkdocs`. Para obtener una lista completa de ejemplos, visite la [documentación de mkdocs](https://squidfunk.github.io/mkdocs-material/reference/).

## Versionado

Para crear versiones, ejecute estos comandos

```bash
mike deploy --update-aliases 1.0 latest
mike set-default latest
mike deploy --update-aliases 2.0 latest
mkdocs serve
```

## Código

```python
print("hello world!")
```

## Código con números de línea

```python linenums="1"
def bubble_sort(items):
    for i in range(len(items)):
        for j in range(len(items) - 1 - i):
            if items[j] > items[j + 1]:
                items[j], items[j + 1] = items[j + 1], items[j]
```

## Código con destacados

```python hl_lines="2 3"
def bubble_sort(items):
    for i in range(len(items)):
        for j in range(len(items) - 1 - i):
            if items[j] > items[j + 1]:
                items[j], items[j + 1] = items[j + 1], items[j]
```

## Código con pestañas

=== "Encabezado"

    ```c
    #include <stdio.h>

    int main(void) {
        printf("Hello world!\n");
        return 0;
    }
    ```

=== "Otro encabezado"

    ```c++
    #include <iostream>

    int main(void) {
        std::cout << "Hello world!" << std::endl;
        return 0;
    }
    ```

## Más tabs

=== "Windows"

    Si estás en windows descargar el archivo `Win32.zip` e instalalo.

=== "MacOS"

    Correr `brew install foo`.

=== "Linux"

    Correr `apt-get install foo`.

## Checklists

* [x] Lorem ipsum dolor sit amet, consectetur adipiscing elit
* [ ] Vestibulum convallis sit amet nisi a tincidunt
    * [x] In hac habitasse platea dictumst

## Agregar un botón

[Lanzar el LAB](https://developer.ibm.com){: .md-button .md-button--primary }

[Visitar IBM Developer](https://developer.ibm.com){: .md-button }

[Registrarse! :fontawesome-solid-paper-plane:](https://cloud.ibm.com){: .md-button .md-button--primary }

## Call outs

!!! tip
    Se puede usar `note`, `abstract`, `info`, `tip`, `success`, `question`
    `warning`, `failure`, `danger`, `bug`, `quote` or `example`.

!!! note
    Una nota.

!!! abstract
    Un abstract.

!!! info
    Alguna info.

!!! success
    Bien.

!!! question
    Pregunta.

!!! warning
    Cuidado.

!!! danger
    Peligro.

!!! example
    Un ejemplo.

!!! bug
    Un error.

## Call outs con código

!!! note
    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod
    nulla. Curabitur feugiat, tortor non consequat finibus, justo purus auctor
    massa, nec semper lorem quam in massa.

    ```python
    def bubble_sort(items):
        for i in range(len(items)):
            for j in range(len(items) - 1 - i):
                if items[j] > items[j + 1]:
                    items[j], items[j + 1] = items[j + 1], items[j]
    ```

    Nunc eu odio eleifend, blandit leo a, volutpat sapien. Phasellus posuere in
    sem ut cursus. Nullam sit amet tincidunt ipsum, sit amet elementum turpis.
    Etiam ipsum quam, mattis in purus vitae, lacinia fermentum enim.

## Formato

Además del típico *cursiva*, y **negrita** ahora está soportado:

* ==resaltado==
* ^^subrayado^^
* ~~tachado~~

## Tablas

| **OS por Aplicación** | **Usuario** | **Password** |
| - | - | - |
| Windows VM | `Administrator` | `foo` |
| Linux VM | `root` | `bar` |

## Emojis

Sí! Esto funciona! :smiley: :+1: