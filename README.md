# Fire TV — Screensaver sem suspensão automática

Tutorial para configurar o Amazon Fire TV para:

- Ativar o protetor de tela após alguns minutos de inatividade.
- Manter o protetor de tela funcionando sem o Fire TV entrar em suspensão após 20 minutos.
- Utilizar até 24 horas de `sleep_timeout`.
- Permitir futuramente testar tempos menores, como 2 minutos.

---

## 1. Objetivo

O Fire TV possui dois comportamentos diferentes:

1. **Screensaver / Protetor de tela**
   - Mostra as imagens da Amazon após determinado período de inatividade.

2. **Suspensão do dispositivo**
   - Depois de um período maior de inatividade, o Fire TV pode sair do screensaver e entrar em estado de suspensão, fazendo a TV ficar sem sinal.

Este procedimento separa os dois temporizadores.

### Configuração final desejada

```text
Inatividade
    ↓
5 minutos
    ↓
Protetor de tela da Amazon
    ↓
Permanece no protetor de tela
    ↓
Sem suspensão automática por até 24 horas
```

---

# 2. Requisitos

- Amazon Fire TV / Fire TV Stick
- Mac, Linux ou Windows
- Android Platform Tools (ADB)
- Fire TV e computador na mesma rede
- Depuração ADB habilitada no Fire TV

---

# 3. Instalar o ADB no macOS

No Mac, utilizando Homebrew:

```bash
brew install android-platform-tools
```

Verifique:

```bash
adb version
```

Deve aparecer algo semelhante a:

```text
Android Debug Bridge version 1.0.41
```

---

# 4. Descobrir o endereço IP do Fire TV

No Fire TV, normalmente:

```text
Configurações
→ Minha Fire TV
→ Sobre
→ Rede
```

Anote o endereço IP.

Exemplo:

```text
192.168.0.18
```

> O IP pode mudar dependendo da configuração da rede. Sempre confirme o endereço atual antes de executar `adb connect`.

---

# 5. Ativar a depuração ADB

No Fire TV:

```text
Configurações
→ Minha Fire TV
→ Sobre
```

Selecione o nome do dispositivo várias vezes para habilitar as opções de desenvolvedor, caso ainda não estejam habilitadas.

Depois:

```text
Opções do desenvolvedor
→ Depuração ADB → LIGADO
```

---

# 6. Conectar o computador ao Fire TV

No Terminal do Mac:

```bash
adb connect 192.168.0.18:5555
```

Substitua `192.168.0.18` pelo IP atual do seu Fire TV.

Na primeira conexão, o Fire TV poderá mostrar uma mensagem solicitando autorização da conexão ADB.

Aceite.

Depois confirme:

```bash
adb devices
```

O resultado deverá conter algo semelhante a:

```text
192.168.0.18:5555    device
```

---

# 7. Desativar a suspensão automática por 24 horas

Este é o comando principal:

```bash
adb shell settings put secure sleep_timeout 86400000
```

O valor:

```text
86400000 ms
```

corresponde a:

```text
24 horas
```

## O que esse comando faz?

Ele aumenta o tempo de `sleep_timeout` do Fire TV para 24 horas.

Isso **não desativa o protetor de tela**.

O protetor de tela continua podendo ser iniciado normalmente pelo seu próprio temporizador.

---

# 8. Configurar o início do protetor de tela

O tempo de início do protetor de tela deve ser configurado pela própria interface do Fire TV.

Caminho:

```text
Configurações
→ Tela e sons
→ Protetor de tela
→ Hora de início
```

Selecione:

```text
5 minutos
```

### Configuração recomendada

```text
Hora de início: 5 minutos
Sleep timeout: 24 horas
```

---

# 9. Resultado esperado

Com as configurações acima:

```text
TV em uso
   ↓
Usuário para de interagir
   ↓
5 minutos
   ↓
Amazon Screensaver é iniciado
   ↓
Screensaver permanece ativo
   ↓
O Fire TV não entra automaticamente em suspensão após 20 minutos
```

O comportamento foi testado utilizando:

```bash
adb shell settings put secure sleep_timeout 86400000
```

Com o protetor de tela configurado para 10 minutos, o screensaver entrou normalmente após aproximadamente 10 minutos.

Depois de ultrapassados os antigos 20 minutos de suspensão, o Fire TV continuou no protetor de tela.

---

# 10. Por que 24 horas?

O Fire TV originalmente estava utilizando:

```text
sleep_timeout = 1200000
```

Isso corresponde a:

```text
20 minutos
```

O problema era que, depois de entrar no screensaver, o Fire TV posteriormente poderia entrar em suspensão.

Alterando para:

```text
86400000
```

o tempo passa para:

```text
24 horas
```

Assim, o screensaver e a suspensão ficam praticamente separados durante o uso diário.

---

# 11. Verificar a configuração

Para verificar o `sleep_timeout`:

```bash
adb shell settings get secure sleep_timeout
```

O resultado esperado:

```text
86400000
```

Também é possível verificar o estado geral de energia:

```bash
adb shell dumpsys power | grep -Ei 'Wakefulness|timeout|Sleep timeout|Screen off timeout|Wake Locks'
```

Quando o protetor de tela estiver funcionando, poderá aparecer:

```text
mWakefulness=Dreaming
```

Isso indica que o sistema está efetivamente no estado de Dream/Screensaver.

---

# 12. Verificar se o Amazon Screensaver está ativo

Com o protetor de tela aparecendo:

```bash
adb shell dumpsys dreams
```

O resultado deverá indicar o serviço do screensaver da Amazon, semelhante a:

```text
mCurrentDreamName=ComponentInfo{
com.amazon.ftv.screensaver/
com.amazon.ftv.screensaver.app.services.ScreensaverService
}
```

Isso confirma que o protetor de tela da Amazon está efetivamente sendo executado.

---

# 13. Opção futura: utilizar 2 minutos

O aplicativo de screensaver do Fire TV contém recursos para:

```text
2 minutos
5 minutos
10 minutos
15 minutos
```

Entretanto, a opção de 2 minutos pode não aparecer na interface gráfica dependendo da versão/região do Fire TV.

Portanto:

### Configuração recomendada

Utilizar:

```text
5 minutos
```

pela própria interface da TV.

### Se no futuro for necessário utilizar 2 minutos

É possível investigar a configuração interna do aplicativo para tentar ativar a opção de 2 minutos.

**Não modificar o APK diretamente sem necessidade.**

O fato de o APK conter a opção de 2 minutos não significa necessariamente que ela possa ser ativada diretamente por uma simples configuração ADB.

---

# 14. Não confundir os dois temporizadores

É importante diferenciar:

```text
screensaver start time
```

de:

```text
sleep_timeout
```

O primeiro determina quando o protetor de tela aparece.

O segundo determina quando o Fire TV pode entrar em suspensão.

Por exemplo:

```text
Screensaver: 5 minutos
Sleep timeout: 24 horas
```

significa:

```text
5 minutos → protetor de tela
24 horas → suspensão automática
```

---

# 15. Comando principal para guardar

Se o objetivo for simplesmente restaurar a configuração de 24 horas:

```bash
adb shell settings put secure sleep_timeout 86400000
```

Este é o comando mais importante deste tutorial.

---

# 16. Se o Fire TV for reiniciado

Um simples reinício normalmente não exige que o comando seja executado novamente.

Depois de reiniciar, entretanto, é recomendável verificar:

```bash
adb shell settings get secure sleep_timeout
```

Se retornar:

```text
86400000
```

a configuração continua aplicada.

---

# 17. Se o Fire TV receber uma atualização

Uma atualização do Fire OS pode eventualmente alterar ou restaurar determinadas configurações.

Depois de uma atualização importante, verifique:

```bash
adb shell settings get secure sleep_timeout
```

Se o resultado não for:

```text
86400000
```

execute novamente:

```bash
adb shell settings put secure sleep_timeout 86400000
```

---

# 18. Depois de uma restauração de fábrica

Uma restauração de fábrica remove as configurações personalizadas.

Nesse caso, será necessário:

1. Configurar novamente o Fire TV.
2. Ativar a depuração ADB.
3. Descobrir o novo endereço IP.
4. Conectar novamente pelo ADB.
5. Executar:

```bash
adb connect IP_DO_FIRE_TV:5555
```

6. Aplicar:

```bash
adb shell settings put secure sleep_timeout 86400000
```

7. Ir para as configurações do Fire TV e definir:

```text
Protetor de tela → Hora de início → 5 minutos
```

---

# 19. Restaurar o comportamento de suspensão original

Antes de alterar qualquer configuração, é recomendável anotar o valor original.

No caso deste procedimento, o valor originalmente utilizado era:

```text
1200000
```

correspondente a:

```text
20 minutos
```

Para restaurar esse valor:

```bash
adb shell settings put secure sleep_timeout 1200000
```

Depois confirme:

```bash
adb shell settings get secure sleep_timeout
```

Resultado esperado:

```text
1200000
```

---

# 20. Configuração rápida

Depois que o ADB já estiver configurado e conectado:

### Manter o Fire TV sem suspensão por 24 horas

```bash
adb shell settings put secure sleep_timeout 86400000
```

### Verificar

```bash
adb shell settings get secure sleep_timeout
```

Esperado:

```text
86400000
```

### Configurar o screensaver

Na TV:

```text
Configurações
→ Tela e sons
→ Protetor de tela
→ Hora de início
→ 5 minutos
```

---

# 21. Resumo

Configuração recomendada:

| Configuração | Valor |
|---|---:|
| Protetor de tela | 5 minutos |
| Sleep timeout | 24 horas |
| `sleep_timeout` | `86400000 ms` |
| Suspensão original testada | 20 minutos |
| `sleep_timeout` original | `1200000 ms` |

### Comando principal

```bash
adb shell settings put secure sleep_timeout 86400000
```

### Verificação

```bash
adb shell settings get secure sleep_timeout
```

Resultado:

```text
86400000
```

---

## ⚠️ Observação

Este procedimento altera configurações internas do Fire TV através do ADB.

Os comandos apresentados não modificam o APK do Amazon Screensaver nem substituem o aplicativo original.

O objetivo é apenas alterar o tempo de suspensão do sistema, mantendo o protetor de tela oficial da Amazon.

A configuração de 2 minutos deve ser considerada uma opção experimental caso ela não esteja disponível na interface gráfica do Fire TV.
