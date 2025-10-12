🌊 1️⃣ Detrend / Remover DC — o que é e por que “evita vazamento em 0 Hz”
➤ O que é “DC” (componente de 0 Hz)

Em sinais, DC (Direct Current) = valor médio constante do sinal.
No caso de sensores inerciais:

    - Acelerômetro: sempre sente gravidade (~9,81 m/s²) + pequenas variações mecânicas.

    - Giroscópio: mesmo parado, costuma ter bias (pequeno desvio constante, ex. +0,1 °/s).

    - Magnetômetro: sempre sente o campo magnético terrestre (~50 µT) fixo no espaço.

Esse “valor constante” é energia no zero da frequência (0 Hz).
Se você fizer a FFT de um sinal com offset, o primeiro ponto (frequência zero) fica muito alto, e isso espalha energia artificialmente para as outras frequências.

➤ Vazamento de energia (“spectral leakage”)

Quando você faz uma FFT de uma janela finita de dados (ex.: 2 s), você está “recortando” o sinal com uma janela (tipo Hanning, rectangular etc.).
Isso, matematicamente, é uma multiplicação no tempo, que vira uma convolução no domínio da frequência.
Ou seja: um componente forte em 0 Hz (DC) espalha um pouquinho de energia para as outras frequências — esse é o vazamento espectral (spectral leakage).

💡 Resumo prático:
Se o seu sinal tem offset (DC) e você faz FFT:

o pico enorme em 0 Hz “vaza” e sujas as vizinhanças,

escondendo picos pequenos reais (como o 1× da rotação).

📊 Então, ao remover o valor médio (detrend), você:

zera o pico em 0 Hz (limpa o espectro),

melhora a definição dos picos reais (1×, 2×, etc.).

⚙️ 2️⃣ O que são picos em 1× e 2×

“1×” e “2×” são frequências harmônicas da rotação da roda (ou eixo).

Termo	Significado	Exemplo (roda a 600 rpm)
1×	1 vez por rotação (fundamental)	600 rpm ÷ 60 = 10 Hz
2×	2 vezes por rotação (harmônico segundo)	20 Hz
3×	3 vezes por rotação	30 Hz

----------------------------------------------------------------------------------------------------------------
🎯 O que é o filtro passa-baixa (low-pass)

Um filtro passa-baixa deixa passar apenas as frequências abaixo de certo limite (cutoff)
e atenua as mais altas, que normalmente são ruído e aliasing waiting to happen.

No caso dos seus sensores (LSM6DSV16X), o acelerômetro e giroscópio amostram a 1920 Hz, ou seja:

O Nyquist (metade da taxa de amostragem) é 960 Hz.

Tudo que existir acima de 960 Hz seria refletido (“dobrado”) para dentro do espectro — o famoso aliasing.

Mesmo abaixo disso, acima de umas centenas de hertz, o sinal tende a ser só ruído eletrônico e vibração estrutural sem relevância física.

⚙️ Por que usar low-pass (filtro passa-baixa)

Eliminar ruído de alta frequência (inútil):

O que interessa no seu caso são as componentes mecânicas da rotação da roda,
que estão na faixa de 0 Hz até, no máximo, 100 Hz.

Tudo acima disso são ruídos eletrônicos, vibrações de estrutura, aliasing, etc.

Preparar para o downsampling (reduzir taxa de amostragem):

Você grava a 1920 Hz, mas só precisa de, por exemplo, 400 Hz para analisar rotação.

Antes de reduzir o número de amostras, você precisa cortar as frequências acima da nova Nyquist (senão ocorre aliasing).

Se você for para 400 Hz, o novo Nyquist é 200 Hz → daí vem o cutoff de 200 Hz.

Melhorar a relação sinal-ruído (SNR):

Removendo ruído de alta frequência, a energia útil (picos 1×, 2×) fica mais clara no PSD.

📊 Por que 200 Hz especificamente?

Vamos raciocinar numericamente:

Fator	Valor típico	Justificativa
ODR (acel/gyro)	1920 Hz	alta demais pro fenômeno estudado
Faixa útil (rotação + harmônicos)	até ~50 Hz (no máximo 100 Hz)	1800 rpm → 30 Hz no 1×, 60 Hz no 2×
Cutoff escolhido	200 Hz	>3× acima da maior frequência útil (pra evitar distorção do sinal), mas bem abaixo de ruído eletrônico e antes do novo Nyquist se fizer downsampling a 400 Hz