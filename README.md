# my version of lifegame as a proof of concept

![image](https://user-images.githubusercontent.com/31476977/114442617-1b9a3e80-9ba3-11eb-8692-0b84cd3b32cc.png)

the code:


import pygame
import numpy as np
import time
pygame.init()


pygame.display.init()

#ancho y alto de la pantalla

width, height = 600, 600

#creating the window
screen = pygame.display.set_mode((height,width))

#background color (black)
bg = 25, 25, 25

screen.fill(bg)

#tamaño de pixeles
nxC, nyC = 50, 50

dimCW = width / nxC 
dimCH = height / nyC

#estado de las celdas. Vivas =1, muertas =0
#matriz con zeros de la cantidad de celdas que tiene el juego
gameState = np.zeros((nxC, nyC))


#creo automatas de inicio
#automata palo
gameState[5, 3] = 1
gameState[5, 4] = 1
gameState[5, 5] = 1

#automata movil
gameState[21, 21] = 1
gameState[22, 22] = 1
gameState[22, 23] = 1
gameState[21, 23] = 1
gameState[20, 23] = 1

# Control de la ejecución - En True se inicia pausado (Para poder ver la forma inicial de los aútomatas):
pauseExec = True

# Controla la finalización del juego:
endGame = False

# Acumulador de cantidad de iteraciones:
iteration = 0

#bucle de ejecucion
while not endGame:
    
    newGameState = np.copy(gameState)
    
    screen.fill(bg)
    
    time.sleep(0.1)

    # Registro de eventos de teclado y mouse
    ev = pygame.event.get()

    # Contador de población:
    population = 0

    for event in ev:

        # Si cierran la ventana finalizo el juego
        if event.type == pygame.QUIT:
            endGame = True
            break

        if event.type == pygame.KEYDOWN:

            # Si tocan escape finalizo el juego
            if event.key == pygame.K_ESCAPE:
                endGame = True
                break

            # Si tocan la tecla r limpio la grilla, reseteo población e iteración y pongo pausa
            if event.key == pygame.K_r:
                iteration = 0
                gameState = np.zeros((nxC, nyC))
                newGameState = np.zeros((nxC, nyC))
                pauseExec = True
            else:
                # Si tocan cualquier tecla no contemplada, pauso o reanudo el juego
                pauseExec = not pauseExec

        # Detección de click del mouse:
        mouseClick = pygame.mouse.get_pressed()

        # Obtención de posición del cursor en la pantalla:
        # Si se hace click con cualquier botón del mouse, se obtiene un valor en mouseClick mayor a cero
        if sum(mouseClick) > 0:

            # Click del medio pausa / reanuda el juego
            if mouseClick[1]:

                pauseExec = not pauseExec

            else:

                # Obtengo las coordenadas del cursor del mouse en pixeles
                posX, posY, = pygame.mouse.get_pos()

                # Convierto de coordenadas en pixeles a celda clickeada en la grilla
                celX, celY = int(np.floor(posX / dimCW)), int(np.floor(posY / dimCH))

                # Click izquierdo y derecho permutan entre vida y muerte
                newGameState[celX, celY] = not gameState[celX, celY]
                #newGameState[celX, celY] = 1
                #newGameState[celX, celY] = not mouseClick[2]

    if not pauseExec:
        # Incremento el contador de generaciones
        iteration += 1
    
            
    for y in range(0, nxC):
        for x in range(0, nyC):
            
            if not pauseExec:
                
                # calculamos el numero de vecinos cercanos
                n_neigh =   (
                            gameState[(x - 1) % nxC, (y - 1) % nyC] + 
                            gameState[(x    ) % nxC, (y - 1) % nyC] + 
                            gameState[(x + 1) % nxC, (y - 1) % nyC] + 
                            gameState[(x - 1) % nxC, (y    ) % nyC] + 
                            gameState[(x + 1) % nxC, (y    ) % nyC] + 
                            gameState[(x - 1) % nxC, (y + 1) % nyC] + 
                            gameState[(x    ) % nxC, (y + 1) % nyC] + 
                            gameState[(x + 1) % nxC, (y + 1) % nyC] 
                )

                # rule 1, una celula muerta con exactamente 3 vecinas vivas, revive
                if gameState[x, y] == 0 and n_neigh == 3:
                    newGameState[x, y] = 1

                # rule 2, una celula viva con menos de 2 o mas de 3 vecinas vivas
                elif gameState[x, y] == 1 and (n_neigh < 2 or n_neigh > 3):
                    newGameState[x, y] = 0
                    
            # Incremento el contador de población:
            if gameState[x, y] == 1:
                population += 1

            # Creamos el poligono de cada celda a dibujar
            poly = [
                        (int(x   *   dimCW), int(y    * dimCH)),
                        (int((x+1) * dimCW), int(y    * dimCH)),
                        (int((x+1) * dimCW), int((y+1)* dimCH)),
                        (int((x)   * dimCW), int((y+1)* dimCH))
                        ]

            # y dibujamos la celda para cada par de x e y 
            if newGameState[x, y] == 0:
                pygame.draw.polygon(screen, (128, 128, 128), poly, 1) #cuadrado pintado de negro
            else:
                if pauseExec:
                    # Con el juego pausado pinto de gris las celdas
                    pygame.draw.polygon(screen, (128, 128, 128), poly, 0)
                else:
                    # Con el juego ejecutándose pinto de blanco las celdas
                    pygame.draw.polygon(screen, (255, 255, 255), poly, 0)
    
    # Actualizo el título de la ventana
    title = f"Juego de la vida - Marcos Soto - Población: {population} - Generación: {iteration}"
    if pauseExec:
        title += " - [PAUSADO]"
    pygame.display.set_caption(title)

    # Actualizo gameState
    gameState = np.copy(newGameState)

    # Muestro y actualizo los fotogramas en cada iteración del bucle principal
    pygame.display.flip()
        


pygame.display.quit()
        
        
    
