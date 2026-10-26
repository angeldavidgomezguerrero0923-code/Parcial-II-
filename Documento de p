"""
main.py
=======
Ejecución del programa y casos de prueba de "Guardians of the Ancient Kingdom".

Dos formas de usar este archivo:
    python main.py            -> demostración: combate de ejemplo + 12 casos de prueba.
    pytest main.py -v         -> pruebas automatizadas (funciones test_*).

Las clases NO se definen aquí: se importan desde personajes.py.
"""

try:
    import pytest
except ImportError:  # Permite `python main.py` sin tener pytest instalado.
    class _PytestFalso:
        """Reemplazo mínimo: solo para que el archivo se pueda importar y correr la demo."""
        class mark:
            @staticmethod
            def parametrize(*args, **kwargs):
                return lambda funcion: funcion
    pytest = _PytestFalso()

from personajes import (
    Arquero,
    AtaqueInvalidoError,
    Combate,
    Guerrero,
    Mago,
    Personaje,
)


# ===========================================================================
# PRUEBAS AUTOMATIZADAS (Pytest)
# ===========================================================================

# --- Caso 1: creación correcta -------------------------------------------
def test_creacion_correcta_de_los_tres_tipos():
    g = Guerrero("Arthas", 100, 30, 20)
    m = Mago("Gandalf", 80, 40, 10)
    a = Arquero("Legolas", 90, 35, 15)
    for p in (g, m, a):
        assert isinstance(p, Personaje)
        assert p.esta_vivo()
    assert (g.nombre, g.vida, g.ataque, g.defensa) == ("Arthas", 100, 30, 20)


def test_personaje_es_abstracto():
    with pytest.raises(TypeError):
        Personaje("X", 10, 10, 10)


# --- Caso 2: vida inválida -----------------------------------------------
@pytest.mark.parametrize("vida", [101, -1, 1000])
def test_rechaza_vida_fuera_de_rango(vida):
    with pytest.raises(ValueError):
        Guerrero("Arthas", vida, 30, 20)


def test_rechaza_vida_de_tipo_incorrecto():
    with pytest.raises(TypeError):
        Guerrero("Arthas", "100", 30, 20)
    with pytest.raises(TypeError):
        Guerrero("Arthas", 50.5, 30, 20)


# --- Caso 3: nombre vacío ------------------------------------------------
@pytest.mark.parametrize("nombre", ["", "   "])
def test_rechaza_nombre_vacio(nombre):
    with pytest.raises(ValueError):
        Mago(nombre, 80, 40, 10)


def test_rechaza_nombre_no_texto():
    with pytest.raises(TypeError):
        Mago(123, 80, 40, 10)


# --- Caso 4: ataque y defensa inválidos ----------------------------------
@pytest.mark.parametrize("ataque", [0, -5])
def test_rechaza_ataque_no_positivo(ataque):
    with pytest.raises(ValueError):
        Arquero("Legolas", 90, ataque, 15)


@pytest.mark.parametrize("defensa", [-1, -20])
def test_rechaza_defensa_negativa(defensa):
    with pytest.raises(ValueError):
        Arquero("Legolas", 90, 35, defensa)


def test_defensa_cero_es_valida():
    assert Arquero("Legolas", 90, 35, 0).defensa == 0


def test_setters_validan_igual_que_el_constructor():
    g = Guerrero("Arthas", 100, 30, 20)
    with pytest.raises(ValueError):
        g.vida = 150
    with pytest.raises(ValueError):
        g.ataque = 0
    with pytest.raises(ValueError):
        g.defensa = -3
    g.vida = 50
    assert g.vida == 50


# --- Caso 5: habilidad del Guerrero --------------------------------------
def test_guerrero_golpe_poderoso():
    guerrero = Guerrero("Arthas", 100, 30, 20)
    objetivo = Guerrero("Dummy", 100, 10, 10)
    assert guerrero.atacar(objetivo) == 26      # 30 * 1.2 - 10
    assert objetivo.vida == 74


def test_guerrero_redondea_hacia_abajo_y_no_hace_danio_negativo():
    guerrero = Guerrero("Arthas", 100, 11, 0)   # 11 * 1.2 = 13.2 -> 13
    assert guerrero.atacar(Mago("M", 100, 5, 0)) == 13
    debil = Guerrero("Debil", 100, 5, 0)        # 6 - 50 < 0 -> 0
    assert debil.atacar(Mago("M2", 100, 5, 50)) == 0


# --- Caso 6: Mago ignora la defensa --------------------------------------
def test_mago_ignora_defensa():
    mago = Mago("Gandalf", 80, 40, 10)
    objetivo = Guerrero("Tanque", 100, 10, 20)
    assert mago.atacar(objetivo) == 40
    assert objetivo.vida == 60
    muralla = Guerrero("Muralla", 100, 10, 500)
    assert mago.atacar(muralla) == 40


# --- Caso 7: disparo crítico del Arquero ---------------------------------
def test_arquero_disparo_critico():
    arquero = Arquero("Legolas", 90, 35, 15)
    objetivo = Guerrero("Dummy", 100, 10, 15)
    assert arquero.atacar(objetivo) == 40       # (35 - 15) * 2
    assert objetivo.vida == 60


def test_arquero_sin_critico_si_no_supera_defensa():
    arquero = Arquero("Legolas", 90, 35, 15)
    assert arquero.atacar(Guerrero("A", 100, 10, 35)) == 0   # ataque == defensa
    assert arquero.atacar(Guerrero("B", 100, 10, 60)) == 0   # ataque < defensa


# --- Caso 8: vida nunca negativa -----------------------------------------
def test_vida_nunca_negativa():
    objetivo = Guerrero("Debil", 20, 10, 0)
    recibido = objetivo.recibir_danio(50)
    assert recibido == 20                       # daño efectivo, no 50
    assert objetivo.vida == 0
    assert not objetivo.esta_vivo()


def test_recibir_danio_rechaza_cantidad_negativa():
    with pytest.raises(ValueError):
        Guerrero("X", 50, 10, 0).recibir_danio(-5)


def test_recibir_danio_no_descuenta_defensa_otra_vez():
    p = Guerrero("X", 100, 10, 50)
    assert p.recibir_danio(30) == 30
    assert p.vida == 70


# --- Caso 9: derrotado no puede atacar -----------------------------------
def test_derrotado_no_puede_atacar():
    caido = Mago("Caido", 10, 40, 0)
    caido.recibir_danio(10)
    with pytest.raises(AtaqueInvalidoError):
        caido.atacar(Guerrero("Vivo", 100, 10, 0))


def test_no_se_puede_atacar_a_un_derrotado():
    atacante = Mago("Atacante", 100, 40, 0)
    caido = Guerrero("Caido", 5, 10, 0)
    caido.recibir_danio(5)
    with pytest.raises(AtaqueInvalidoError):
        atacante.atacar(caido)


def test_objetivo_invalido():
    mago = Mago("Gandalf", 80, 40, 10)
    with pytest.raises(TypeError):
        mago.atacar("no soy un personaje")
    with pytest.raises(TypeError):
        mago.atacar(None)
    with pytest.raises(ValueError):
        mago.atacar(mago)


# --- Casos 10 y 11: combate completo y ganador ---------------------------
def test_combate_completo_del_enunciado():
    arthas = Guerrero("Arthas", 100, 30, 20)
    gandalf = Mago("Gandalf", 80, 40, 10)
    combate = Combate(arthas, gandalf)
    assert combate.iniciar(verbose=False) == Combate.VICTORIA
    assert combate.ganador is gandalf
    assert arthas.vida == 0
    assert gandalf.vida == 2
    assert combate.rondas == 3                  # 6 turnos = 3 rondas


def test_iniciar_dos_veces_no_altera_el_resultado():
    combate = Combate(Guerrero("Arthas", 100, 30, 20), Mago("Gandalf", 80, 40, 10))
    resultado = combate.iniciar(verbose=False)
    rondas = combate.rondas
    assert combate.iniciar(verbose=False) == resultado
    assert combate.rondas == rondas == 3


def test_combate_gana_el_primero_si_derrota_antes():
    fuerte = Mago("Fuerte", 100, 99, 0)
    debil = Guerrero("Debil", 10, 5, 0)
    combate = Combate(fuerte, debil)
    combate.iniciar(verbose=False)
    assert combate.ganador is fuerte
    assert debil.vida == 0


def test_combate_valida_participantes():
    p = Mago("A", 50, 10, 0)
    with pytest.raises(TypeError):
        Combate(p, "otro")
    with pytest.raises(ValueError):
        Combate(p, p)
    muerto = Guerrero("Muerto", 1, 5, 0)
    muerto.recibir_danio(1)
    with pytest.raises(ValueError):
        Combate(p, muerto)


def test_cambiar_turno_y_verificar_ganador():
    a = Guerrero("A", 100, 30, 0)
    b = Mago("B", 100, 30, 0)
    combate = Combate(a, b)
    assert combate.verificar_ganador() is None
    combate.cambiar_turno()
    b.recibir_danio(100)
    assert combate.verificar_ganador() is a


# --- Caso 12: empate técnico ---------------------------------------------
def test_empate_tecnico_sin_dano():
    a = Guerrero("A", 100, 5, 50)    # 6 - 50 -> 0 de daño
    b = Arquero("B", 100, 5, 50)     # 5 no supera 50 -> 0 de daño
    combate = Combate(a, b)
    assert combate.iniciar(verbose=False) == Combate.EMPATE_TECNICO
    assert combate.ganador is None
    assert a.vida == 100 and b.vida == 100


def test_empate_por_limite_de_50_rondas():
    # A causa 1 de daño por turno y B no causa nada: en 50 rondas B pierde 50 de vida.
    a = Guerrero("A", 100, 1, 50)    # 1 * 6 // 5 = 1 de daño contra defensa 0
    b = Arquero("B", 100, 5, 0)      # 5 no supera la defensa 50 de A -> 0 de daño
    combate = Combate(a, b)
    assert combate.iniciar(verbose=False) == Combate.EMPATE_RONDAS
    assert combate.rondas == 50
    assert b.vida == 50


def test_polimorfismo_en_combate():
    # Combate no distingue tipos: acepta cualquier combinación.
    for t1 in (Guerrero, Mago, Arquero):
        for t2 in (Guerrero, Mago, Arquero):
            c = Combate(t1("P1", 100, 30, 10), t2("P2", 100, 30, 10))
            assert c.iniciar(verbose=False) in (
                Combate.VICTORIA, Combate.EMPATE_TECNICO, Combate.EMPATE_RONDAS
            )


# ===========================================================================
# DEMOSTRACIÓN (python main.py)
# ===========================================================================
def _intentar(descripcion, funcion):
    """Ejecuta una operación que debe fallar y muestra el error capturado."""
    try:
        funcion()
        print(f"  [FALLO] {descripcion}: NO lanzó error")
    except (TypeError, ValueError, AtaqueInvalidoError) as error:
        print(f"  [OK] {descripcion} -> {type(error).__name__}: {error}")


def demostracion():
    print("#" * 60)
    print("CASO 1: Creación correcta de los tres tipos")
    print("#" * 60)
    guerrero = Guerrero("Arthas", 100, 30, 20)
    mago = Mago("Gandalf", 80, 40, 10)
    arquero = Arquero("Legolas", 90, 35, 15)
    for p in (guerrero, mago, arquero):
        print(" ", p.mostrar_estado())

    print("\nCASOS 2, 3 y 4: Validaciones de datos")
    _intentar("Vida 101", lambda: Guerrero("X", 101, 30, 20))
    _intentar("Vida -5", lambda: Guerrero("X", -5, 30, 20))
    _intentar("Nombre vacío", lambda: Mago("", 80, 40, 10))
    _intentar("Ataque 0", lambda: Arquero("X", 90, 0, 15))
    _intentar("Defensa -1", lambda: Arquero("X", 90, 35, -1))

    print("\nCASO 5: Guerrero (30 de ataque vs 10 de defensa -> esperado 26)")
    dummy = Guerrero("Dummy", 100, 10, 10)
    print("  Daño causado:", guerrero.atacar(dummy), "| Vida del objetivo:", dummy.vida)

    print("\nCASO 6: Mago ignora defensa (40 de ataque vs 20 de defensa -> esperado 40)")
    dummy = Guerrero("Dummy", 100, 10, 20)
    print("  Daño causado:", mago.atacar(dummy), "| Vida del objetivo:", dummy.vida)

    print("\nCASO 7: Arquero crítico (35 vs 15 -> base 20, esperado 40)")
    dummy = Guerrero("Dummy", 100, 10, 15)
    print("  Daño causado:", arquero.atacar(dummy), "| Vida del objetivo:", dummy.vida)

    print("\nCASO 8: La vida nunca es negativa (20 de vida, 50 de daño)")
    debil = Guerrero("Debil", 20, 10, 0)
    recibido = debil.recibir_danio(50)
    print(f"  Daño efectivo: {recibido} | Vida final: {debil.vida} | Vivo: {debil.esta_vivo()}")

    print("\nCASO 9: Un derrotado no puede atacar (ni ser atacado)")
    _intentar("Derrotado ataca", lambda: debil.atacar(Mago("Vivo", 100, 10, 0)))
    _intentar("Atacar a derrotado", lambda: mago.atacar(debil))
    _intentar("Objetivo inválido", lambda: mago.atacar("no soy personaje"))

    print("\nCASOS 10 y 11: Combate completo y ganador (ejemplo del enunciado)")
    arthas = Guerrero("Arthas", 100, 30, 20)
    gandalf = Mago("Gandalf", 80, 40, 10)
    combate = Combate(arthas, gandalf)
    combate.iniciar()
    print(f"  Ganador esperado: Gandalf | Ganador obtenido: {combate.ganador.nombre}")

    print("\nCASO 12: Empate técnico (ninguno puede hacer daño)")
    a = Guerrero("Guerrero débil", 100, 5, 50)
    b = Arquero("Arquero débil", 100, 5, 50)
    combate = Combate(a, b)
    combate.iniciar()
    print(f"  Rondas jugadas: {combate.rondas} | Resultado: {combate.resultado}")


if __name__ == "__main__":
    demostracion()
