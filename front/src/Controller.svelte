<script>
    import { refreshToken, accessToken, spotifyAPIHandler, username, isAuthValid, gameScore } from "./lib/stores";
    import { MAX_ROUNDS, destroySdk } from "./lib/utils";
    import LoadAuth from "./lib/LoadAuth.svelte";
    import GamePlay from "./game/GamePlay.svelte";
    import LandingContent from "./landing/LandingContent.svelte";
    import PreGameScreen from "./pregame/PreGameScreen.svelte";

    export let showFullCopyright = false;
    
    let currentMode = "landing";    // landing | pregame | game
    let nextMode = currentMode;
    
    let gameInfo = {
        maxRounds: 5,
        content: {},
        played: []
    }

    function goToSelection() {
        if ($refreshToken && $accessToken) {
            $spotifyAPIHandler.setAccessToken($accessToken);
            nextMode = "pregame";
            currentMode = "";
        }
    }

    function goToGame(event) {
        $gameScore = 0;
        gameInfo = {
            maxRounds: MAX_ROUNDS,
            content: event.detail.content,
            played: []
        }
        console.log(gameInfo.content);
        nextMode = "game";
        currentMode = "";
    }

    function resetAll() {
        $refreshToken = null;
        $username = null;
        $isAuthValid = null;
        goToLanding();
    }

    function goToLanding() {
        nextMode = "landing";
        currentMode = nextMode;
    }

    function update() {
        console.log("update");
        currentMode = nextMode;
    }

    // dynamic content
    $: showFullCopyright = currentMode === "landing";
    $: if (currentMode !== "game") destroySdk();
</script>

<LoadAuth />

{#if currentMode === "landing"}
    <LandingContent 
        on:ready={goToSelection}
        on:reset={resetAll}
        on:outroend={update}
    />

{:else if currentMode === "pregame"}
    <PreGameScreen 
        on:reset={goToLanding}
        on:submit={goToGame}
        on:outroend={update}   
    />

{:else if currentMode === "game"}
    <GamePlay 
        gameInfo={gameInfo}
        on:newGame={goToSelection}
        on:playAgain={goToGame}
        on:outroend={update}
    />

{/if}