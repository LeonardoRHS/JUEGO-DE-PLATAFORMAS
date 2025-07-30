# Salto Extremo

## 🎮 Descripción del juego

el juego esta intencionado a tener un nivel de dificultad normal gracias a los diferentes tipos de plataformas usadas y a su posicionamiento dentro de los niveles del juego y La mecanica principal del juego es recolectar anillos con la intención de alcanzar un número establecido de anillos para poder desbloquear la puerta para pasar al siguiente nivel.

## Caracteristicas

 Sistema de recolección de anillos: el jugador debe recolectar 5 anillos para desbloquear las puertas.

 Sistema de guardado y carga: recuerda anillos recolectados y la posición del jugador

 plataformas variadas(fija, oscilatoria, caida, fragil, salto)

 Efectos de sonido para salto, recolección y muerte.

## 🧩 Assets utilizados

fondo de nivel 1

[fondo.cielo (1).zip](https://github.com/user-attachments/files/21518238/fondo.cielo.1.zip)

fondo de nivel 2

[infierno.zip](https://github.com/user-attachments/files/21518225/infierno.zip)

personaje

[personaje (1).zip](https://github.com/user-attachments/files/21518242/personaje.1.zip)

plataformas

[plataforma (1).zip](https://github.com/user-attachments/files/21518244/plataforma.1.zip)


puerta

[puerta de fuego (1).zip](https://github.com/user-attachments/files/21518253/puerta.de.fuego.1.zip)


anillos

[anillo.zip](https://github.com/user-attachments/files/21518251/anillo.zip)


🎵 Sonidos:

sonido de muerte

[sonidodemuerte1.zip](https://github.com/user-attachments/files/21518259/sonidodemuerte1.zip)

sonido de anillo

[sonidodeanillo.zip](https://github.com/user-attachments/files/21518256/sonidodeanillo.zip)


sonido de salto

[sonido de salto.zip](https://github.com/user-attachments/files/21518266/sonido.de.salto.zip)


PERSONAJE

```gdscript

extends CharacterBody2D

# --- Variables del jugador ---
var vida : int = 100                    # Vida del jugador (no utilizada en este código, pero útil para ampliaciones futuras)
var puntaje : int = 0                   # Contador de anillos recolectados
var velocidad : int = 200              # Velocidad de movimiento horizontal
var gravedad : int = 1000              # Fuerza de gravedad aplicada al jugador
var fuerza_salto : int = -400          # Fuerza del salto (valor negativo para subir)

# --- Referencias a nodos ---
@onready var label_puntaje : Label = $"../UI_Puntaje"     # Etiqueta en la interfaz que muestra los anillos
@onready var jump_sound: AudioStreamPlayer2D = $jump_sound # Sonido que se reproduce al saltar

# --- Función que se ejecuta al iniciar ---
func _ready():
    actualizar_label()                  # Muestra el puntaje actual al comenzar
    add_to_group("jugador")            # Agrega este nodo al grupo "jugador" (útil para identificarlo desde otros scripts)

# --- Movimiento del personaje y controles ---
func _physics_process(delta):
    # Aplica gravedad si el jugador no está en el suelo
    if not is_on_floor():
        velocity.y += gravedad * delta

    # Movimiento horizontal con teclas izquierda/derecha
    var direccion = Input.get_axis("ui_left", "ui_right")
    velocity.x = direccion * velocidad

    # Salto si el jugador está en el suelo y presiona la tecla de salto
    if Input.is_action_just_pressed("ui_up") and is_on_floor():
        velocity.y = fuerza_salto
        jump_sound.play()  # Reproduce sonido de salto

    # Mueve al personaje usando física integrada de Godot
    move_and_slide()

    # Guardado y carga con teclas personalizadas (deben estar en el Input Map)
    if Input.is_action_pressed("guardar"):
        guardar_datos()
    if Input.is_action_pressed("cargar"):
        cargar_datos()

# --- Función para sumar anillos y actualizar etiqueta ---
func sumar_puntos(cantidad: int):
    puntaje += cantidad
    actualizar_label()  # Refresca el texto del HUD

# --- Actualiza el texto del HUD con el puntaje actual ---
func actualizar_label():
    label_puntaje.text = "anillos: %d" % [puntaje]

# --- Detecta colisiones con otros cuerpos (como enemigos o picos) ---
func _on_body_entered(body):
    if body.is_in_group("jugador"):
        body.call("morir")  # Si el cuerpo es del grupo jugador, llama a su función de morir

# --- Función para reiniciar la escena cuando el jugador muere ---
func morir():
    get_tree().reload_current_scene()  # Recarga el nivel actual desde cero

# --- Guarda la posición y puntaje en un archivo JSON ---
func guardar_datos():
    var datos = {
        "jugador": {
            "puntaje": puntaje,
            "posicion": {
                "x": "%.8f" % global_position.x,  # Guarda la posición X con precisión
                "y": "%.8f" % global_position.y   # Guarda la posición Y con precisión
            }
        }
    }

    var json_texto = JSON.stringify(datos, "\t")  # Convierte el diccionario a texto JSON
    var archivo = FileAccess.open("res://juego_guardado.json", FileAccess.WRITE)
    archivo.store_string(json_texto)
    archivo.close()
    print("Todo salió bien, archivo guardado")

# --- Carga los datos del archivo JSON y los aplica al jugador ---
func cargar_datos():
    if not FileAccess.file_exists("res://juego_guardado.json"):
        print("No hay archivo")
        return

    var archivo = FileAccess.open("res://juego_guardado.json", FileAccess.READ)
    var json_caracter = archivo.get_as_text()
    archivo.close()

    var json = JSON.new()
    var error = json.parse(json_caracter)
    if error != OK:
        print("No se parseó", json.get_error_message())
        return

    var datos = json.get_data()

    # Elimina las monedas existentes (si están en el grupo "monedas")
    for moneda in get_tree().get_nodes_in_group("monedas"):
        moneda.queue_free()

    # Establece la posición del jugador y su puntaje
    global_position = Vector2(
        float(datos["jugador"]["posicion"]["x"]),
        float(datos["jugador"]["posicion"]["y"])
    )

    puntaje = datos["jugador"]["puntaje"]
    actualizar_label()

...
```

PUERTA

```gdscript

extends Area2D

# --- Referencia al Label que muestra el mensaje "Nivel bloqueado" ---
@onready var label_bloqueado: Label = $"LabelBloqueado"

# --- Se ejecuta al iniciar la escena ---
func _ready():
    label_bloqueado.visible = false  # Oculta el mensaje al comenzar
    body_entered.connect(_on_body_entered)  # Conecta la señal de entrada de cuerpos al área

# --- Función que se ejecuta cuando un cuerpo entra en el área ---
func _on_body_entered(body):
    if body.is_in_group("jugador"):  # Verifica si el cuerpo que entró es el jugador
        if body.puntaje >= 5:
            # Si el jugador tiene 5 o más anillos, cambia al siguiente nivel
            get_tree().change_scene_to_file("res://NIVEL2/escenas/NIVEL2.tscn")  # Cambia esta ruta si es otro nivel
        else:
            # Si no tiene suficientes anillos, muestra el mensaje de "bloqueado" durante 2 segundos
            label_bloqueado.visible = true
            await get_tree().create_timer(2).timeout  # Espera 2 segundos
            label_bloqueado.visible = false  # Oculta el mensaje después de esperar

...
```

ANILLOS

```gdscript

extends Area2D

# --- Referencia al sonido que se reproduce al recolectar el anillo ---
@onready var ring_sfx: AudioStreamPlayer2D = $ring_sfx

# --- Valor en puntos (anillos) que otorga esta moneda ---
var valor_puntos : int = 1

# --- Función que se ejecuta al iniciar la escena ---
func _ready():
    ring_sfx.stop()  # Asegura que el sonido esté apagado al iniciar

    # Conecta la señal 'body_entered' para detectar colisiones con el jugador
    body_entered.connect(_on_body_entered)

    # Añade este nodo al grupo "monedas", útil para eliminar todas si se recarga la escena
    add_to_group("monedas")

# --- Se ejecuta cuando un cuerpo entra en el área de la moneda ---
func _on_body_entered(body : Node):
    if body.is_in_group("jugador"):  # Verifica que quien tocó la moneda es el jugador
        body.sumar_puntos(valor_puntos)  # Llama a la función del jugador que suma puntos
        ring_sfx.play()  # Reproduce el sonido de recolectar
        await get_tree().create_timer(0.2).timeout  # Espera 0.2 segundos para que el sonido se escuche bien
        queue_free()  # Elimina la moneda de la escena

...
```

PLATAFORMAS

```gdscript

extends Area2D

# --- Tipos de plataformas disponibles ---
enum TipoPlataforma {FIJA, OSCILATORIA, FRAGIL, REBOTE, CAIDA}

# --- Parámetros configurables desde el editor ---
@export var type: TipoPlataforma = TipoPlataforma.FIJA         # Tipo de plataforma (por defecto, fija)
@export var direccion := "vertical"                             # Dirección de movimiento para plataformas oscilatorias
@export var fuerza_rebote := 2.0                                # Multiplicador de salto para plataformas rebote
@export var desplazamiento := 100                               # Distancia de oscilación
@export var duracion := 2.0                                     # Tiempo de ida o vuelta en la oscilación
@onready var jump_from: AudioStreamPlayer = $jump_from          # Sonido al rebotar

# --- Variables internas ---
var posicion_inicial := Vector2.ZERO  # Guarda la posición original para las plataformas móviles

# --- Al iniciar la escena ---
func _ready():
    posicion_inicial = position                # Guarda la posición inicial
    actualizar_plataforma()                    # Configura color y comportamiento según el tipo
    monitorable = true                         # Permite detectar colisiones con el jugador
    monitoring = true

# --- Aplica color y comportamiento según el tipo de plataforma ---
func actualizar_plataforma():
    match type:
        TipoPlataforma.FIJA:
            $Sprite2D.modulate = Color.NAVY_BLUE
        TipoPlataforma.OSCILATORIA:
            $Sprite2D.modulate = Color.DARK_ORANGE
            oscilar()  # Inicia movimiento oscilatorio
        TipoPlataforma.FRAGIL:
            $Sprite2D.modulate = Color.NAVY_BLUE
        TipoPlataforma.REBOTE:
            $Sprite2D.modulate = Color.YELLOW
        TipoPlataforma.CAIDA:
            $Sprite2D.modulate = Color.NAVY_BLUE

# --- Detecta cuando el jugador pisa la plataforma ---
func _on_body_entered(body: Node2D) -> void:
    if body.is_in_group("jugador"):
        match type:
            TipoPlataforma.FRAGIL:
                # Plataforma desaparece 0.25 segundos después de pisarla
                await get_tree().create_timer(0.25).timeout
                queue_free()
            
            TipoPlataforma.REBOTE:
                # Si el jugador tiene método especial para rebotar, lo usamos
                if body.has_method("puede_rebotar"):
                    jump_from.play()
                    body.puede_rebotar(fuerza_rebote)
                else:
                    # Si no, lo impulsamos manualmente hacia arriba
                    body.velocity.y = body.fuerza_salto * fuerza_rebote
            
            TipoPlataforma.CAIDA:
                # Plataforma cae hacia abajo al ser pisada
                var tween = create_tween()
                tween.tween_property(self, "position:y", position.y + 200, 1.0)

# --- Aplica movimiento oscilatorio continuo a la plataforma ---
func oscilar():
    var tween = create_tween()
    tween.set_loops()  # Hace que el movimiento se repita infinitamente

    if direccion == "vertical":
        # Sube y baja
        tween.tween_property(self, "position:y", posicion_inicial.y + desplazamiento, duracion).set_trans(Tween.TRANS_SINE).set_ease(Tween.EASE_IN_OUT)
        tween.tween_property(self, "position:y", posicion_inicial.y - desplazamiento, duracion).set_trans(Tween.TRANS_SINE).set_ease(Tween.EASE_IN_OUT)
    elif direccion == "horizontal":
        # Se mueve de izquierda a derecha
        tween.tween_property(self, "position:x", posicion_inicial.x + desplazamiento, duracion).set_trans(Tween.TRANS_SINE).set_ease(Tween.EASE_IN_OUT)
        tween.tween_property(self, "position:x", posicion_inicial.x - desplazamiento, duracion).set_trans(Tween.TRANS_SINE).set_ease(Tween.EASE_IN_OUT)

...
```


ZONA DE ELIMINACION

```gdscript

extends Area2D

# Referencia al sonido que se reproduce al morir
@onready var death_sound: AudioStreamPlayer = $death_sound

# Función que se llama cuando un cuerpo entra en esta área (pico, trampa, enemigo)
func _on_body_entered(body: Node2D):
    if body.is_in_group("jugador"):  # Verifica que sea el jugador
        death_sound.play()            # Reproduce el sonido de muerte
        body.set_physics_process(false)  # Detiene el movimiento del jugador
        # Espera a que termine el sonido antes de reiniciar la escena
        await get_tree().create_timer(death_sound.stream.get_length()).timeout
        get_tree().reload_current_scene()  # Reinicia el nivel
        # Alternativamente, si quieres eliminar al jugador en lugar de reiniciar escena:
        # body.queue_free()

...
```
