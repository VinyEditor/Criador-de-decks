# Criador-de-decks
Criador de decks para pokemon

import requests
import random
from collections import Counter

# API base do Pokémon TCG (pokemontcg.io)
BASE_URL = "https://api.pokemontcg.io/v2/cards"

# Função para buscar cartas com base em critérios
def buscar_cartas(tipo=None, set_code=None, name=None, supertype=None, limit=100):
    params = {"pageSize": limit}
    if tipo:
        params["types"] = tipo
    if set_code:
        params["set.id"] = set_code
    if name:
        params["name"] = name
    if supertype:
        params["supertype"] = supertype  # Ex: "Pokémon", "Trainer", "Energy"
    response = requests.get(BASE_URL, params=params)
    if response.status_code == 200:
        return response.json()["data"]
    else:
        print(f"Erro na busca: {response.status_code}")
        return []

# Função para selecionar Pokémons chave e do tipo
def selecionar_pokemons(tipo_pokemon, pokemons_chave, sets):
    pokemons = []
    
    # Adicionar Pokémons chave
    for pokemon in pokemons_chave:
        cartas = buscar_cartas(name=pokemon, set_code=random.choice(sets))
        if cartas:
            pokemons.append(cartas[0])
    
    # Adicionar Pokémons do tipo especificado (até 10)
    cartas_tipo = buscar_cartas(tipo=tipo_pokemon, supertype="Pokémon", set_code=random.choice(sets), limit=20)
    for carta in cartas_tipo:
        if carta not in pokemons and len(pokemons) < 20:  # Limite para não sobrecarregar
            pokemons.append(carta)
    
    return pokemons

# Função para adicionar treinadores/itens
def adicionar_treinadores(intencao, sets, quantidade=10):
    treinadores = []
    
    if intencao == "regional":
        # Focar em itens competitivos (ex: aceleração, cura)
        nomes_competitivos = ["Potion", "Switch", "Professor's Research", "Rare Candy"]
        for nome in nomes_competitivos:
            cartas = buscar_cartas(name=nome, supertype="Trainer", set_code=random.choice(sets), limit=5)
            treinadores.extend(cartas[:2])  # Até 2 cópias por item
    elif intencao == "casual":
        # Adicionar variedade (ex: itens temáticos ou aleatórios)
        cartas_aleatorias = buscar_cartas(supertype="Trainer", set_code=random.choice(sets), limit=20)
        treinadores.extend(random.sample(cartas_aleatorias, quantidade))
    
    return treinadores

# Função para adicionar energias básicas
def adicionar_energias(tipo_pokemon, quantidade=15):
    energias = []
    # Buscar energias básicas do tipo
    cartas_energia = buscar_cartas(tipo=tipo_pokemon, supertype="Energy", limit=quantidade)
    if cartas_energia:
        energias.extend(cartas_energia)
    else:
        # Fallback para energia básica genérica (ex: Basic Energy)
        cartas_basic = buscar_cartas(name="Basic Energy", limit=quantidade)
        energias.extend(cartas_basic)
    
    return energias

# Função para balancear o deck (garantir 60 cartas, limites de cópias)
def balancear_deck(cartas):
    deck = []
    contador = Counter()
    
    for carta in cartas:
        nome = carta["name"]
        if contador[nome] < 4:  # Máximo 4 cópias por carta
            deck.append(carta)
            contador[nome] += 1
    
    # Preencher até 60 cartas duplicando aleatoriamente
    while len(deck) < 60:
        carta_aleatoria = random.choice(deck)
        nome = carta_aleatoria["name"]
        if contador[nome] < 4:
            deck.append(carta_aleatoria)
            contador[nome] += 1
    
    return deck[:60]

# Função principal para criar o deck
def criar_deck_completo(tipo_pokemon, pokemons_chave, intencao, sets=["sv6", "sv7"]):  # sv6: Twilight Masquerade, sv7: Ascended Heroes
    # Selecionar Pokémons
    pokemons = selecionar_pokemons(tipo_pokemon, pokemons_chave, sets)
    
    # Adicionar treinadores
    treinadores = adicionar_treinadores(intencao, sets)
    
    # Adicionar energias
    energias = adicionar_energias(tipo_pokemon)
    
    # Combinar todas as cartas
    todas_cartas = pokemons + treinadores + energias
    
    # Balancear o deck
    deck_balanceado = balancear_deck(todas_cartas)
    
    # Retornar detalhes do deck (nome, tipo, raridade, etc.)
    return [{"name": c["name"], "type": c.get("types", ["N/A"]), "rarity": c.get("rarity", "N/A"), "set": c["set"]["name"]} for c in deck_balanceado]

# Exemplo de uso
tipo = "Fire"  # Tipo de Pokémon (ex: Fire, Water, Grass)
pokemons_chave = ["Charizard", "Pidgeot"]  # Lista de Pokémons chave
intencao = "regional"  # "regional" ou "casual"
deck_sugerido = criar_deck_completo(tipo, pokemons_chave, intencao)

print("Deck sugerido (60 cartas):")
for i, carta in enumerate(deck_sugerido, 1):
    print(f"{i}. {carta['name']} - Tipo: {carta['type']} - Raridade: {carta['rarity']} - Set: {carta['set']}")

# Contagem final
contador_final = Counter([c["name"] for c in deck_sugerido])
print("\nContagem de cartas:")
for nome, qtd in contador_final.items():
    print(f"{nome}: {qtd}")
