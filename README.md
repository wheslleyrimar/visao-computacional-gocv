# Visão Computacional com GoCV

Repositório no Github: 

https://github.com/wheslleyrimar/visao-computacional-gocv

### Pacote principal

```go
package main

```

Declara que este é o pacote principal. No Go, um programa executável deve ter um pacote principal chamado `main`. Isso indica ao compilador onde iniciar a execução do programa.

### Importações

```go
import (
	"fmt"
	"image/color"
	"gocv.io/x/gocv"
)

```

Importa três pacotes:

- `fmt`: pacote da biblioteca padrão do Go, usado para funções de formatação de entrada/saída, como imprimir mensagens na tela.
- `image/color`: também da biblioteca padrão, define tipos e funções para trabalhar com cores.
- `gocv.io/x/gocv`: pacote da biblioteca GoCV, que fornece funções para manipulação de imagens e visão computacional.

### Função principal

```go
func main() {

```

Define a função `main`, que é o ponto de entrada do programa. Quando o programa é executado, a função `main` é chamada primeiro.

### Configuração do dispositivo de captura de vídeo

```go
    deviceID := 1

```

Define `deviceID` como `1`. Esse valor é o identificador do dispositivo de captura de vídeo (normalmente uma webcam). No Go, o `:=` é um operador de declaração curta que cria e inicializa a variável `deviceID` com o valor `1`.

### Abrindo a webcam

```go
	webcam, err := gocv.OpenVideoCapture(deviceID)
	if err != nil {
		fmt.Println(err)
		return
	}
	defer webcam.Close()

```

1. `webcam, err := gocv.OpenVideoCapture(deviceID)`:
    - `gocv.OpenVideoCapture(deviceID)`: chama a função `OpenVideoCapture` da biblioteca GoCV com `deviceID` como argumento. Tenta abrir o dispositivo de captura de vídeo (webcam).
    - `webcam`: variável que armazena o objeto de captura de vídeo.
    - `err`: variável que armazena qualquer erro que possa ocorrer ao tentar abrir o dispositivo.
2. `if err != nil { ... }`:
    - Verifica se `err` não é `nil`, o que indicaria que ocorreu um erro ao abrir a webcam.
    - `fmt.Println(err)`: imprime o erro na tela.
    - `return`: encerra a execução da função `main` se ocorreu um erro.
3. `defer webcam.Close()`:
    - `defer`: garante que a função `webcam.Close()` será chamada ao final da função `main`, mesmo se ocorrer um erro antes. `webcam.Close()` libera os recursos associados ao dispositivo de captura de vídeo.

### Abrindo a janela de exibição

```go
	window := gocv.NewWindow("Face Detect")
	defer window.Close()

```

1. `window := gocv.NewWindow("Face Detect")`:
    - `gocv.NewWindow("Face Detect")`: chama a função `NewWindow` da biblioteca GoCV, criando uma nova janela com o título "Face Detect".
    - `window`: variável que armazena o objeto janela.
2. `defer window.Close()`:
    - Garante que a função `window.Close()` será chamada ao final da função `main`, liberando os recursos associados à janela.

### Preparando a matriz de imagem

```go
	img := gocv.NewMat()
	defer img.Close()

```

1. `img := gocv.NewMat()`:
    - `gocv.NewMat()`: chama a função `NewMat` da biblioteca GoCV, criando uma nova matriz de imagem vazia.
    - `img`: variável que armazena a matriz de imagem.
2. `defer img.Close()`:
    - Garante que a função `img.Close()` será chamada ao final da função `main`, liberando os recursos associados à matriz de imagem.

### Definindo a cor do retângulo

```go
	blue := color.RGBA{0, 0, 255, 0}

```

1. `color.RGBA{0, 0, 255, 0}`:
    - Cria uma cor usando o modelo RGBA (Red, Green, Blue, Alpha).
    - `0, 0, 255, 0`: define os valores de vermelho (0), verde (0), azul (255) e alpha (0). `255` para azul significa que a cor é azul puro.
2. `blue := color.RGBA{0, 0, 255, 0}`:
    - Atribui a cor azul à variável `blue`.

### Carregando o classificador de faces

```go
	classifier := gocv.NewCascadeClassifier()
	defer classifier.Close()

	if !classifier.Load("data/haarcascade_frontalface_default.xml") {
		fmt.Println("Error reading cascade file: data/haarcascade_frontalface_default.xml")
		return
	}

```

1. `classifier := gocv.NewCascadeClassifier()`:
    - `gocv.NewCascadeClassifier()`: chama a função `NewCascadeClassifier` da biblioteca GoCV, criando um novo objeto classificador de cascata.
    - `classifier`: variável que armazena o objeto classificador.
2. `defer classifier.Close()`:
    - Garante que a função `classifier.Close()` será chamada ao final da função `main`, liberando os recursos associados ao classificador.
3. `if !classifier.Load("data/haarcascade_frontalface_default.xml") { ... }`:
    - `classifier.Load("data/haarcascade_frontalface_default.xml")`: chama a função `Load` do classificador, carregando o arquivo XML com os dados do classificador de faces.
    - `!classifier.Load(...)`: verifica se o carregamento falhou (`Load` retorna `false` se não conseguir carregar o arquivo).
    - `fmt.Println("Error reading cascade file: data/haarcascade_frontalface_default.xml")`: imprime uma mensagem de erro na tela.
    - `return`: encerra a execução da função `main` se ocorrer um erro ao carregar o arquivo.

### Explicação simples do classificador

Um classificador de cascata, como o usado aqui, é um algoritmo de visão computacional treinado para reconhecer padrões específicos, como faces humanas. Ele utiliza uma série de filtros (ou "cascatas") que são aplicados à imagem para identificar regiões que contêm o padrão desejado (neste caso, faces). O arquivo XML carregado (`haarcascade_frontalface_default.xml`) contém os dados do modelo treinado, que o classificador usa para detectar faces em uma imagem.

### Loop de leitura da câmera e detecção de faces

```go
	fmt.Printf("start reading camera device: %v\\n", deviceID)
	for {
		if ok := webcam.Read(&img); !ok {
			fmt.Printf("cannot read device %v\\n", deviceID)
			return
		}
		if img.Empty() {
			continue
		}

		// detect faces
		rects := classifier.DetectMultiScale(img)
		fmt.Printf("found %d faces\\n", len(rects))

		// draw a rectangle around each face on the original image
		for _, r := range rects {
			gocv.Rectangle(&img, r, blue, 3)
		}

		// show the image in the window, and wait 1 millisecond
		window.IMShow(img)
		window.WaitKey(1)
	}
}

```

1. `fmt.Printf("start reading camera device: %v\\n", deviceID)`:
    - `fmt.Printf(...)`: função que formata e imprime uma mensagem na tela. Aqui, imprime uma mensagem indicando que a leitura da câmera foi iniciada, incluindo o `deviceID`.
2. `for { ... }`:
    - Inicia um loop infinito. O loop continuará até que o programa seja interrompido manualmente ou uma condição de saída seja atingida dentro do loop.
3. `if ok := webcam.Read(&img); !ok { ... }`:
    - `webcam.Read(&img)`: chama a função `Read` do objeto `webcam`, tentando capturar um frame da câmera e armazená-lo na matriz `img`. `&img` passa um ponteiro para `img`.
    - `ok := webcam.Read(&img)`: atribui o resultado da leitura (um booleano) à variável `ok`.
    - `if !ok { ... }`: verifica se `ok` é `false`, o que indica que a leitura falhou.
    - `fmt.Printf("cannot read device %v\\n", deviceID)`: imprime uma mensagem de erro.
    - `return`: encerra a execução da função `main`.
4. `if img.Empty() { continue }`:
    - `img.Empty()`: chama a função `Empty` da matriz `img` para verificar se a imagem está vazia.
    - `if img.Empty() { continue }`: se a imagem estiver vazia, pula para a próxima iteração do loop.
5. `rects := classifier.DetectMultiScale(img)`:
    - `classifier.DetectMultiScale(img)`: chama a função `DetectMultiScale` do classificador para detectar faces na imagem `img`. Retorna um slice de retângulos (`rects`), onde cada retângulo representa a posição de uma face detectada.
6. `fmt.Printf("found %d faces\\n", len(rects))`:
    - `fmt.Printf(...)`: imprime o número de faces detectadas (comprimento do slice `

rects`).

1. `for _, r := range rects { gocv.Rectangle(&img, r, blue, 3) }`:
    - `for _, r := range rects { ... }`: itera sobre cada retângulo `r` em `rects`.
    - `gocv.Rectangle(&img, r, blue, 3)`: chama a função `Rectangle` da biblioteca GoCV para desenhar um retângulo azul (`blue`) de espessura `3` ao redor da face detectada na imagem `img`, nas coordenadas do retângulo `r`.
2. `window.IMShow(img)`:
    - `window.IMShow(img)`: chama a função `IMShow` da janela para exibir a imagem `img` na janela.
3. `window.WaitKey(1)`:
    - `window.WaitKey(1)`: chama a função `WaitKey` da janela, esperando `1` milissegundo por uma tecla pressionada. Isso permite que a janela seja atualizada e processa eventos da interface gráfica.

Em resumo, este código captura continuamente frames da webcam, detecta faces nesses frames, desenha retângulos ao redor das faces detectadas e exibe os frames na janela. A utilização de `defer` garante que todos os recursos (webcam, janela, imagem, classificador) serão corretamente liberados ao final da execução.