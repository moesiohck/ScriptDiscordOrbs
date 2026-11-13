> [!CAUTION]
> A partir de 7 de abril de 2026, o Discord expressou sua intenção de reprimir a automatização da conclusão de missões.
>
> Alguns usuários receberam a seguinte mensagem do sistema:
>
> <img width="836" height="272" alt="image" src="https://gist.github.com/user-attachments/assets/6b439f4b-4381-4524-8540-b6a4777a80d0" />
> Não há muito que eu possa fazer para tornar o script indetectável, então use-o por sua conta e risco, pois você provavelmente SERÁ sinalizado ao fazê-lo.

# Complete Recent Discord Quest (Versão PT-BR por @Nephestox) Atualizado 05/11/2026

Script com traduções, melhorias visuais e uma função para parar a execução de forma segura.

> [!NOTE]
> Este script não funciona no navegador para missões que exigem que você jogue um game! Use o [aplicativo para desktop](https://discord.com/download) para completar essas missões.

---

## Modificações desta Versão (por @Nephestox)

Esta versão do script mantém toda a funcionalidade principal, adicionando melhorias de usabilidade e controle:

### Principais Novidades ✨

- **Função de Parada Segura (`pararMissao()`):** Agora você pode interromper o script a qualquer momento de forma segura. Ao digitar `pararMissao()` no console, a execução é finalizada e todas as modificações feitas no cliente do Discord são revertidas, limpando o script completamente.
- **Tradução para Português-Brasil:** Todas as mensagens e logs exibidos no console foram traduzidos para facilitar o entendimento.
- **Melhorias Visuais no Console:** A saída de texto no console foi redesenhada com uma cor ciano (`#58F0F7`), fonte maior e sombra no texto, garantindo uma legibilidade muito superior.

---

<details>
	<summary>Clique para visualizar o Script</summary>
	
```js
delete window.$;

let wpRequire = window.webpackChunkdiscord_app.push([[Symbol()], {}, r => r]);
window.webpackChunkdiscord_app.pop();

function findStore(fn) {
    for (let id in wpRequire.c) {
        let m = wpRequire.c[id]?.exports;
        if (!m) continue;

        try {
            if (Object.prototype.toString.call(m) !== '[object IntlMessagesProxy]' && fn(m)) return m;
        } catch (e) {}

        if (typeof m === 'object' || typeof m === 'function') {
            for (let key of Object.keys(m)) {
                try {
                    let val = m[key];
                    if (val && Object.prototype.toString.call(val) !== '[object IntlMessagesProxy]' && fn(val)) return val;
                } catch (e) {}
            }
        }
    }
    return null;
}

let ApplicationStreamingStore = findStore(s => typeof s?.getStreamerActiveStreamMetadata === 'function');
let RunningGameStore = findStore(s => typeof s?.getRunningGames === 'function' && typeof s?.getGameForPID === 'function');
let QuestsStore = findStore(s => (typeof s?.getQuest === 'function' || typeof s?.getQuests === 'function') && s?.quests != null);
let ChannelStore = findStore(s => typeof s?.getAllThreadsForParent === 'function' || typeof s?.getSortedPrivateChannels === 'function');
let GuildChannelStore = findStore(s => typeof s?.getSFWDefaultChannel === 'function' || typeof s?.getAllGuilds === 'function');
let FluxDispatcher = findStore(s => typeof s?.flushWaitQueue === 'function' || (typeof s?.dispatch === 'function' && typeof s?.subscribe === 'function'));
let rawApi = findStore(s => typeof s?.get === 'function' && typeof s?.post === 'function' && typeof s?.del === 'function' && typeof s?.put === 'function' && typeof s?.patch === 'function' && !s?.Request && !s?.getXHR)
    || Object.values(wpRequire.c).find(x => x?.exports?.Bo?.get || x?.exports?.default?.get)?.exports?.Bo;

// Fallbacks de busca direta
if (!QuestsStore) QuestsStore = Object.values(wpRequire.c).find(x => x?.exports?.A?.__proto__?.getQuest || x?.exports?.default?.__proto__?.getQuest || x?.exports?.Z?.__proto__?.getQuest)?.exports?.A;
if (!RunningGameStore) RunningGameStore = Object.values(wpRequire.c).find(x => x?.exports?.Ay?.getRunningGames || x?.exports?.default?.getRunningGames || x?.exports?.Z?.getRunningGames)?.exports?.Ay;
if (!ApplicationStreamingStore) ApplicationStreamingStore = Object.values(wpRequire.c).find(x => x?.exports?.A?.__proto__?.getStreamerActiveStreamMetadata || x?.exports?.default?.getStreamerActiveStreamMetadata)?.exports?.A;
if (!FluxDispatcher) FluxDispatcher = Object.values(wpRequire.c).find(x => x?.exports?.h?.__proto__?.flushWaitQueue || x?.exports?.default?.__proto__?.flushWaitQueue)?.exports?.h;
if (!rawApi) rawApi = Object.values(wpRequire.c).find(x => x?.exports?.Bo?.get || x?.exports?.default?.get)?.exports?.Bo;

let api = {
    get: (opts) => {
        if (typeof opts === 'string') opts = { url: opts };
        if (rawApi?.Request) return rawApi.get(opts.url);
        return rawApi.get(opts);
    },
    post: (opts) => {
        if (typeof opts === 'string') opts = { url: opts, body: {} };
        if (rawApi?.Request) {
            let req = rawApi.post(opts.url);
            if (opts.body) req = req.send(opts.body);
            return req;
        }
        return rawApi.post(opts);
    }
};

const missingStores = [];
if (!ApplicationStreamingStore) missingStores.push("ApplicationStreamingStore");
if (!RunningGameStore) missingStores.push("RunningGameStore");
if (!QuestsStore) missingStores.push("QuestsStore");
if (!FluxDispatcher) missingStores.push("FluxDispatcher");
if (!rawApi) missingStores.push("api");

if (missingStores.length > 0) {
    console.warn("%c[Aviso] Módulos ausentes ou ainda não carregados na memória: " + missingStores.join(", "), "color: #ffa502; font-size: 14px; font-weight: bold;");
    console.log("%cDica: Certifique-se de estar usando o aplicativo Desktop do Discord e de ter aberto as Missões (Configurações > Inventário de Presentes / Missões) antes de executar o script.", "color: #58F0F7; font-size: 13px;");
}

window.scriptExecutando = true;
window.unsubscribeFn = null; 

window.pararMissao = function () {
    window.scriptExecutando = false;
    console.log(
        "%cScript parado pelo usuário!",
        "color: #58F0F7; font-size: 18px; font-weight: bold; text-shadow: -1px 0 black, 0 1px black, 1px 0 black, 0 -1px black;",
    );
    console.log(
        "%cO Script foi totalmente limpo do seu Discord, pode voltar a usar normalmente.",
        "color: #58F0F7; font-size: 18px; font-weight: bold; text-shadow: -1px 0 black, 0 1px black, 1px 0 black, 0 -1px black;",
    );
    if (typeof window.realGetRunningGames !== "undefined" && RunningGameStore) RunningGameStore.getRunningGames = window.realGetRunningGames;
    if (typeof window.realGetGameForPID !== "undefined" && RunningGameStore) RunningGameStore.getGameForPID = window.realGetGameForPID;
    if (typeof window.realFunc !== "undefined" && ApplicationStreamingStore) ApplicationStreamingStore.getStreamerActiveStreamMetadata = window.realFunc;
    if (window.unsubscribeFn && FluxDispatcher) FluxDispatcher.unsubscribe("QUESTS_SEND_HEARTBEAT_SUCCESS", window.unsubscribeFn);
};

console.log(
    "%c[Nephestox] Digite 'pararMissao()' no console para parar o script a qualquer momento 👊",
    "color: #58F0F7; font-size: 16px; font-weight: bold; text-shadow: -1px 0 black, 0 1px black, 1px 0 black, 0 -1px black;",
);

const supportedTasks = ["WATCH_VIDEO", "PLAY_ON_DESKTOP", "STREAM_ON_DESKTOP", "PLAY_ACTIVITY", "WATCH_VIDEO_ON_MOBILE"];

const questList = QuestsStore?.quests instanceof Map 
    ? [...QuestsStore.quests.values()] 
    : (Array.isArray(QuestsStore?.quests) ? QuestsStore.quests : Object.values(QuestsStore?.quests || {}));

console.log(`%c[Diagnóstico] Total de missões brutas encontradas no Discord: ${questList.length}`, "color: #58F0F7; font-size: 14px; font-weight: bold;");

let quests = questList.filter(x => 
    !x.userStatus?.completedAt && 
    new Date(x.config?.expiresAt).getTime() > Date.now() && 
    supportedTasks.find(y => Object.keys((x.config?.taskConfig ?? x.config?.taskConfigV2)?.tasks ?? {}).includes(y))
);

console.log(`%c[Diagnóstico] Missões pendentes para processar: ${quests.length}`, "color: #58F0F7; font-size: 14px; font-weight: bold;");

let isApp = typeof DiscordNative !== "undefined";

if (quests.length === 0) {
    if (questList.length === 0) {
        console.log(
            "%c[!] O Discord ainda não carregou as missões na memória!\nAbra as Configurações de Usuário > Inventário de Presentes / Missões para forçar o Discord a carregar as missões e execute o script novamente.",
            "color: #ffa502; font-size: 15px; font-weight: bold;",
        );
    } else {
        console.log(
            "%cVocê não tem nenhuma missão pendente! Todas as encontradas já foram concluídas ou expiraram.",
            "color: #58F0F7; font-size: 18px; font-weight: bold; text-shadow: -1px 0 black, 0 1px black, 1px 0 black, 0 -1px black;",
        );
    }
} else {
    let doJob = async function() {
        if (!window.scriptExecutando) return;

        const quest = quests.pop();
        if (!quest) {
            console.log("%cTodas as missões foram finalizadas com sucesso! 🚀", "color: #58F0F7; font-size: 18px; font-weight: bold; text-shadow: -1px 0 black, 0 1px black, 1px 0 black, 0 -1px black;");
            return;
        }

        const questName = quest.config.messages.questName;
        
        if (!quest.userStatus?.enrolledAt && api?.post) {
            console.log(`%cInscrevendo automaticamente na missão: ${questName}...`, "color: #58F0F7; font-size: 14px;");
            try {
                await api.post({url: `/quests/${quest.id}/enroll`, body: { location: 2 }});
            } catch (err) {
                console.log(`%cNão foi possível auto-inscrever em ${questName}, tentando continuar...`, "color: #ffa502; font-size: 12px;");
            }
        }

        const pid = Math.floor(Math.random() * 30000) + 1000;
        const taskConfig = quest.config.taskConfig ?? quest.config.taskConfigV2;
        const taskName = supportedTasks.find(x => taskConfig.tasks[x] != null);
        const taskData = taskConfig.tasks[taskName];
        const applicationId = quest.config.application?.id ?? taskData?.applications?.[0]?.id ?? null;
        const secondsNeeded = taskData.target;
        
        let secondsDone = quest.userStatus?.progress?.[taskName]?.value ?? 0;

        if (taskName === "WATCH_VIDEO" || taskName === "WATCH_VIDEO_ON_MOBILE") {
            const speed = 7;
            let completed = false;
            
            let fn = async () => {			
                while (window.scriptExecutando) {
                    const remaining = Math.min(speed, secondsNeeded - secondsDone);
                    await new Promise(resolve => setTimeout(resolve, remaining * 1000));

                    const timestamp = secondsDone + speed;
                    const res = await api.post({url: `/quests/${quest.id}/video-progress`, body: {timestamp: Math.min(secondsNeeded, timestamp + Math.random())}});
                    completed = res.body.completed_at != null;
                    secondsDone = Math.min(secondsNeeded, timestamp);

                    if (timestamp >= secondsNeeded || !window.scriptExecutando) break;
                }
                if (!completed && window.scriptExecutando) {
                    await api.post({url: `/quests/${quest.id}/video-progress`, body: {timestamp: secondsNeeded}});
                }
                if (window.scriptExecutando) {
                    console.log("%cMissão de vídeo concluída!", "color: #58F0F7; font-size: 18px; font-weight: bold; text-shadow: -1px 0 black, 0 1px black, 1px 0 black, 0 -1px black;");
                    doJob();
                }
            };
            fn();
            console.log(`%cSimulando vídeo para: ${questName}`, "color: #58F0F7; font-size: 18px; font-weight: bold; text-shadow: -1px 0 black, 0 1px black, 1px 0 black, 0 -1px black;");
            
        } else if (taskName === "PLAY_ON_DESKTOP") {
            if (!isApp) {
                console.log(`%cUse o app desktop do Discord para a missão: ${questName}!`, "color: #58F0F7; font-size: 18px; font-weight: bold; text-shadow: -1px 0 black, 0 1px black, 1px 0 black, 0 -1px black;");
                doJob();
            } else {
                api.get({url: `/applications/public?application_ids=${applicationId}`}).then(res => {
                    const appData = res.body[0];
                    const exeName = appData.executables?.find(x => x.os === "win32")?.name?.replace(">","") ?? appData.name.replace(/[\/\\:*?"<>|]/g, "");
                    
                    const fakeGame = {
                        cmdLine: `C:\\Program Files\\${appData.name}\\${exeName}`, exeName,
                        exePath: `c:/program files/${appData.name.toLowerCase()}/${exeName}`,
                        hidden: false, isLauncher: false, id: applicationId, name: appData.name, pid: pid,
                        pidPath: [pid], processName: appData.name, start: Date.now(),
                    };
                    
                    const realGames = RunningGameStore.getRunningGames();
                    const fakeGames = [fakeGame];
                    
                    window.realGetRunningGames = RunningGameStore.getRunningGames;
                    window.realGetGameForPID = RunningGameStore.getGameForPID;
                    
                    RunningGameStore.getRunningGames = () => fakeGames;
                    RunningGameStore.getGameForPID = (pidParam) => fakeGames.find(x => x.pid === pidParam);
                    FluxDispatcher.dispatch({type: "RUNNING_GAMES_CHANGE", removed: realGames, added: [fakeGame], games: fakeGames});
                    
                    let fn = data => {
                        if (!window.scriptExecutando) return;
                        
                        let progress = quest.config.configVersion === 1 ? data.userStatus.streamProgressSeconds : Math.floor(data.userStatus.progress.PLAY_ON_DESKTOP.value);
                        console.log(`%cProgresso (${appData.name}): ${progress}/${secondsNeeded} segundos`, "color: #58F0F7; font-size: 18px; font-weight: bold; text-shadow: -1px 0 black, 0 1px black, 1px 0 black, 0 -1px black;");
                        
                        if (progress >= secondsNeeded) {
                            console.log("%cMissão concluída!", "color: #58F0F7; font-size: 18px; font-weight: bold; text-shadow: -1px 0 black, 0 1px black, 1px 0 black, 0 -1px black;");
                            
                            RunningGameStore.getRunningGames = window.realGetRunningGames;
                            RunningGameStore.getGameForPID = window.realGetGameForPID;
                            FluxDispatcher.dispatch({type: "RUNNING_GAMES_CHANGE", removed: [fakeGame], added: [], games: []});
                            FluxDispatcher.unsubscribe("QUESTS_SEND_HEARTBEAT_SUCCESS", fn);
                            
                            doJob();
                        }
                    };
                    window.unsubscribeFn = fn;
                    FluxDispatcher.subscribe("QUESTS_SEND_HEARTBEAT_SUCCESS", fn);
                    console.log(`%cJogo simulado: ${appData.name}. Aguarde ~${Math.ceil((secondsNeeded - secondsDone) / 60)} min.`, "color: #58F0F7; font-size: 18px; font-weight: bold; text-shadow: -1px 0 black, 0 1px black, 1px 0 black, 0 -1px black;");
                });
            }
        } else if (taskName === "STREAM_ON_DESKTOP") {
            if (!isApp) {
                console.log(`%cUse o app desktop do Discord para a missão: ${questName}!`, "color: #58F0F7; font-size: 18px; font-weight: bold; text-shadow: -1px 0 black, 0 1px black, 1px 0 black, 0 -1px black;");
                doJob();
            } else {
                window.realFunc = ApplicationStreamingStore.getStreamerActiveStreamMetadata;
                ApplicationStreamingStore.getStreamerActiveStreamMetadata = () => ({id: applicationId, pid, sourceName: null});
                
                let fn = data => {
                    if (!window.scriptExecutando) return;
                    
                    let progress = quest.config.configVersion === 1 ? data.userStatus.streamProgressSeconds : Math.floor(data.userStatus.progress.STREAM_ON_DESKTOP.value);
                    console.log(`%cProgresso da stream (Alvo): ${progress}/${secondsNeeded} segundos`, "color: #58F0F7; font-size: 18px; font-weight: bold; text-shadow: -1px 0 black, 0 1px black, 1px 0 black, 0 -1px black;");
                    
                    if (progress >= secondsNeeded) {
                        console.log("%cMissão concluída!", "color: #58F0F7; font-size: 18px; font-weight: bold; text-shadow: -1px 0 black, 0 1px black, 1px 0 black, 0 -1px black;");
                        
                        ApplicationStreamingStore.getStreamerActiveStreamMetadata = window.realFunc;
                        FluxDispatcher.unsubscribe("QUESTS_SEND_HEARTBEAT_SUCCESS", fn);
                        
                        doJob();
                    }
                };
                window.unsubscribeFn = fn;
                FluxDispatcher.subscribe("QUESTS_SEND_HEARTBEAT_SUCCESS", fn);
                console.log(`%cTransmissão simulada para o jogo alvo. Transmita qualquer janela em call por ~${Math.ceil((secondsNeeded - secondsDone) / 60)} min.`, "color: #58F0F7; font-size: 18px; font-weight: bold; text-shadow: -1px 0 black, 0 1px black, 1px 0 black, 0 -1px black;");
            }
        } else if (taskName === "PLAY_ACTIVITY") {
            const channelId = ChannelStore?.getSortedPrivateChannels?.()?.[0]?.id ?? Object.values(GuildChannelStore?.getAllGuilds?.() ?? {}).find(x => x != null && x.VOCAL?.length > 0)?.VOCAL?.[0]?.channel?.id;
            const streamKey = `call:${channelId}:1`;
            
            let fn = async () => {
                console.log(`%cCompletando a missão: ${questName}`, "color: #58F0F7; font-size: 18px; font-weight: bold; text-shadow: -1px 0 black, 0 1px black, 1px 0 black, 0 -1px black;");
                
                while (window.scriptExecutando) {
                    const res = await api.post({url: `/quests/${quest.id}/heartbeat`, body: {stream_key: streamKey, terminal: false}});
                    const progress = res.body.progress.PLAY_ACTIVITY.value;
                    console.log(`%cProgresso da Atividade: ${progress}/${secondsNeeded} segundos`, "color: #58F0F7; font-size: 18px; font-weight: bold; text-shadow: -1px 0 black, 0 1px black, 1px 0 black, 0 -1px black;");
                    
                    await new Promise(resolve => setTimeout(resolve, 20 * 1000));
                    
                    if (progress >= secondsNeeded || !window.scriptExecutando) {
                        if (window.scriptExecutando) {
                            await api.post({url: `/quests/${quest.id}/heartbeat`, body: {stream_key: streamKey, terminal: true}});
                        }
                        break;
                    }
                }
                if (window.scriptExecutando) {
                    console.log("%cMissão concluída!", "color: #58F0F7; font-size: 18px; font-weight: bold; text-shadow: -1px 0 black, 0 1px black, 1px 0 black, 0 -1px black;");
                    doJob();
                }
            };
            fn();
        }
    };
    doJob();
}
```
</details>

## Como usar este script:

1.  Aceite uma missão em `Descobrir -> Missões`.
2.  Pressione <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>I</kbd> para abrir o DevTools.
3.  Vá para a aba `Console`.
4.  Cole o código desta versão e pressione Enter.
    * (Se não conseguir colar, pode ser necessário digitar `allow pasting` e pressionar Enter primeiro).
    * Uma mensagem aparecerá informando que você pode digitar `pararMissao()` para parar o script.
5.  Siga as instruções que aparecerem no console, dependendo do tipo de missão:
    * Se a missão for "jogar" ou "assistir a um vídeo", você não precisa fazer mais nada, apenas esperar.
    * Se a missão for "transmitir" o jogo, entre em um canal de voz com um amigo (ou uma conta alternativa) e transmita qualquer janela.
6.  Aguarde o tempo necessário para que a missão seja concluída. Você pode acompanhar o progresso pelos logs no console (`Progresso da missão em segundos...`).
7.  Pronto! Agora você pode resgatar sua recompensa.

## FAQ (Perguntas Frequentes)

**P: Posso ser banido por usar isso?**

R: Sempre existe um risco, embora até agora ninguém tenha sido banido por isso ou por coisas semelhantes, como client mods.

**P: Ctrl + Shift + I não funciona.**

R: Você pode baixar o [cliente PTB](https://discord.com/api/downloads/distributions/app/installers/latest?channel=ptb&platform=win&arch=x64) para habilitar o DevTools facilmente.

**P: Ctrl + Shift + I tira uma screenshot.**

R: Desative o atalho no seu aplicativo AMD Radeon.

**P: Recebo um erro de sintaxe/token inesperado.**

R: Certifique-se de que seu navegador não está traduzindo automaticamente a página antes de copiar o script. Desative extensões de tradução e tente novamente.

**P: Estou no Vesktop, mas ele me diz que estou usando um navegador.**

R: Vesktop não é um verdadeiro cliente desktop, é um wrapper de navegador. Baixe o aplicativo desktop oficial.

**P: Posso completar missões expiradas com isso?**

R: Não, não há como fazer isso.

**P: Você pode fazer o script aceitar a missão/recompensa automaticamente?**

R: Não. Ambas as ações podem exigir a resolução de um captcha, então automatizá-las não é uma boa ideia.

## License (GPL-3.0)

<details>
<summary>Clique para expandir</summary>

```txt
GNU GENERAL PUBLIC LICENSE
                       Version 3, 29 June 2007

 Copyright (C) 2026 Nephestox
 Everyone is permitted to copy and distribute verbatim copies
 of this license document, but changing it is not allowed.

    Complete Recent Discord Quest (PT-BR)
    Copyright (C) 2026 Nephestox

    This program is free software: you can redistribute it and/or modify
    it under the terms of the GNU General Public License as published by
    the Free Software Foundation, either version 3 of the License, or
    (at your option) any later version.

    This program is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.

    You should have received a copy of the GNU General Public License
    along with this program.  If not, see [https://www.gnu.org/licenses/](https://www.gnu.org/licenses/).
```

</details>

---
