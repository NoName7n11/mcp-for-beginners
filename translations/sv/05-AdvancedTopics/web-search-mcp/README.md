<!--
CO_OP_TRANSLATOR_METADATA:
{
  "original_hash": "7a11a5dcf2f9fdf6392f5a4545cf005e",
  "translation_date": "2025-06-11T15:47:00+00:00",
  "source_file": "05-AdvancedTopics/web-search-mcp/README.md",
  "language_code": "sv"
}
-->
# Lesson: Bygga en Web Search MCP Server

Det här kapitlet visar hur man bygger en verklig AI-agent som integreras med externa API:er, hanterar olika datatyper, hanterar fel och orkestrerar flera verktyg – allt i ett produktionsklart format. Du kommer att se:

- **Integration med externa API:er som kräver autentisering**
- **Hantering av olika datatyper från flera endpoints**
- **Robust felhantering och loggningsstrategier**
- **Orkestrering av flera verktyg i en enda server**

I slutet kommer du ha praktisk erfarenhet av mönster och bästa praxis som är viktiga för avancerade AI- och LLM-drivna applikationer.

## Introduktion

I den här lektionen lär du dig att bygga en avancerad MCP-server och klient som utökar LLM:s kapabiliteter med realtidsdata från webben via SerpAPI. Detta är en viktig färdighet för att utveckla dynamiska AI-agenter som kan hämta uppdaterad information från webben.

## Läromål

I slutet av denna lektion kommer du kunna:

- Integrera externa API:er (som SerpAPI) säkert i en MCP-server
- Implementera flera verktyg för webb-, nyhets-, produkt-sökning och Q&A
- Tolka och formatera strukturerad data för LLM-användning
- Hantera fel och API-begränsningar effektivt
- Bygga och testa både automatiserade och interaktiva MCP-klienter

## Web Search MCP Server

Den här delen introducerar arkitekturen och funktionerna i Web Search MCP Server. Du får se hur FastMCP och SerpAPI används tillsammans för att utöka LLM:s kapabiliteter med realtidsdata från webben.

### Översikt

Denna implementation innehåller fyra verktyg som visar MCP:s förmåga att hantera olika, externa API-drivna uppgifter på ett säkert och effektivt sätt:

- **general_search**: För breda webbresultat
- **news_search**: För senaste nyhetsrubriker
- **product_search**: För e-handelsdata
- **qna**: För frågor och svar-snippets

### Funktioner
- **Kodexempel**: Inkluderar språk-specifika kodblock för Python (och enkelt utbyggbart till andra språk) med vikbara sektioner för tydlighet

<details>  
<summary>Python</summary>  

```python
# Example usage of the general_search tool
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

async def run_search():
    server_params = StdioServerParameters(
        command="python",
        args=["server.py"],
    )
    async with stdio_client(server_params) as (reader, writer):
        async with ClientSession(reader, writer) as session:
            await session.initialize()
            result = await session.call_tool("general_search", arguments={"query": "open source LLMs"})
            print(result)
```
</details>

Innan du kör klienten är det bra att förstå vad servern gör. [`server.py`](../../../../05-AdvancedTopics/web-search-mcp/server.py) file implements the MCP server, exposing tools for web, news, product search, and Q&A by integrating with SerpAPI. It handles incoming requests, manages API calls, parses responses, and returns structured results to the client.

You can review the full implementation in [`server.py`](../../../../05-AdvancedTopics/web-search-mcp/server.py).

Här är ett kort exempel på hur servern definierar och registrerar ett verktyg:

<details>  
<summary>Python Server</summary> 

```python
# server.py (excerpt)
from mcp.server import MCPServer, Tool

async def general_search(query: str):
    # ...implementation...

server = MCPServer()
server.add_tool(Tool("general_search", general_search))

if __name__ == "__main__":
    server.run()
```
</details>

- **Integration med externa API:er**: Visar säker hantering av API-nycklar och externa anrop
- **Tolkning av strukturerad data**: Visar hur API-svar omvandlas till format som är lätt för LLM att använda
- **Felhantering**: Robust felhantering med lämplig loggning
- **Interaktiv klient**: Inkluderar både automatiserade tester och ett interaktivt läge för testning
- **Kontexthantering**: Använder MCP Context för loggning och spårning av förfrågningar

## Förutsättningar

Innan du börjar, se till att din miljö är korrekt uppsatt genom att följa dessa steg. Detta säkerställer att alla beroenden är installerade och att dina API-nycklar är korrekt konfigurerade för smidig utveckling och testning.

- Python 3.8 eller högre
- SerpAPI API-nyckel (Registrera dig på [SerpAPI](https://serpapi.com/) – gratis nivå finns)

## Installation

För att komma igång, följ dessa steg för att sätta upp din miljö:

1. Installera beroenden med uv (rekommenderas) eller pip:

```bash
# Using uv (recommended)
uv pip install -r requirements.txt

# Using pip
pip install -r requirements.txt
```

2. Skapa en `.env`-fil i projektets rot med din SerpAPI-nyckel:

```
SERPAPI_KEY=your_serpapi_key_here
```

## Användning

Web Search MCP Server är huvudkomponenten som exponerar verktyg för webb-, nyhets-, produkt-sökning och Q&A genom integration med SerpAPI. Den hanterar inkommande förfrågningar, sköter API-anrop, tolkar svar och returnerar strukturerade resultat till klienten.

Du kan granska hela implementationen i [`server.py`](../../../../05-AdvancedTopics/web-search-mcp/server.py).

### Köra servern

För att starta MCP-servern, använd följande kommando:

```bash
python server.py
```

Servern kommer att köras som en stdio-baserad MCP-server som klienten kan ansluta till direkt.

### Klientlägen

Klienten (`client.py`) supports two modes for interacting with the MCP server:

- **Normal mode**: Runs automated tests that exercise all the tools and verify their responses. This is useful for quickly checking that the server and tools are working as expected.
- **Interactive mode**: Starts a menu-driven interface where you can manually select and call tools, enter custom queries, and see results in real time. This is ideal for exploring the server's capabilities and experimenting with different inputs.

You can review the full implementation in [`client.py`](../../../../05-AdvancedTopics/web-search-mcp/client.py).

### Köra klienten

För att köra automatiserade tester (detta startar automatiskt servern):

```bash
python client.py
```

Eller kör i interaktivt läge:

```bash
python client.py --interactive
```

### Testa med olika metoder

Det finns flera sätt att testa och interagera med verktygen som servern tillhandahåller, beroende på dina behov och arbetsflöde.

#### Skriva egna testscripts med MCP Python SDK
Du kan också bygga egna testscripts med MCP Python SDK:

<details>
<summary>Python</summary>

```python
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

async def test_custom_query():
    server_params = StdioServerParameters(
        command="python",
        args=["server.py"],
    )
    
    async with stdio_client(server_params) as (reader, writer):
        async with ClientSession(reader, writer) as session:
            await session.initialize()
            # Call tools with your custom parameters
            result = await session.call_tool("general_search", 
                                           arguments={"query": "your custom query"})
            # Process the result
```
</details>

I detta sammanhang betyder "testscript" ett eget Python-program som du skriver för att agera som klient till MCP-servern. Istället för att vara ett formellt enhetstest låter detta script dig programmässigt ansluta till servern, anropa dess verktyg med valfria parametrar och granska resultaten. Denna metod är användbar för:
- Prototyping och experiment med verktygsanrop
- Validering av hur servern svarar på olika indata
- Automatisering av upprepade verktygsanrop
- Bygga egna arbetsflöden eller integrationer ovanpå MCP-servern

Du kan använda testscripts för att snabbt prova nya frågor, felsöka verktygsbeteenden eller som en utgångspunkt för mer avancerad automation. Nedan är ett exempel på hur du använder MCP Python SDK för att skapa ett sådant script:

## Verktygsbeskrivningar

Du kan använda följande verktyg som tillhandahålls av servern för att utföra olika typer av sökningar och frågor. Varje verktyg beskrivs nedan med dess parametrar och exempel på användning.

Denna sektion ger detaljer om varje tillgängligt verktyg och deras parametrar.

### general_search

Utför en generell webbsökning och returnerar formaterade resultat.

**Så här anropar du verktyget:**

Du kan anropa `general_search` från ditt eget script med MCP Python SDK, eller interaktivt med Inspector eller det interaktiva klientläget. Här är ett kodexempel med SDK:

<details>
<summary>Python Exempel</summary>

```python
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

async def run_general_search():
    server_params = StdioServerParameters(
        command="python",
        args=["server.py"],
    )
    async with stdio_client(server_params) as (reader, writer):
        async with ClientSession(reader, writer) as session:
            await session.initialize()
            result = await session.call_tool("general_search", arguments={"query": "latest AI trends"})
            print(result)
```
</details>

Alternativt, i interaktivt läge, välj `general_search` from the menu and enter your query when prompted.

**Parameters:**
- `query` (string): Sökfrågan

**Exempel på förfrågan:**

```json
{
  "query": "latest AI trends"
}
```

### news_search

Söker efter senaste nyhetsartiklar relaterade till en fråga.

**Så här anropar du verktyget:**

Du kan anropa `news_search` från ditt eget script med MCP Python SDK, eller interaktivt med Inspector eller det interaktiva klientläget. Här är ett kodexempel med SDK:

<details>
<summary>Python Exempel</summary>

```python
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

async def run_news_search():
    server_params = StdioServerParameters(
        command="python",
        args=["server.py"],
    )
    async with stdio_client(server_params) as (reader, writer):
        async with ClientSession(reader, writer) as session:
            await session.initialize()
            result = await session.call_tool("news_search", arguments={"query": "AI policy updates"})
            print(result)
```
</details>

Alternativt, i interaktivt läge, välj `news_search` from the menu and enter your query when prompted.

**Parameters:**
- `query` (string): Sökfrågan

**Exempel på förfrågan:**

```json
{
  "query": "AI policy updates"
}
```

### product_search

Söker efter produkter som matchar en fråga.

**Så här anropar du verktyget:**

Du kan anropa `product_search` från ditt eget script med MCP Python SDK, eller interaktivt med Inspector eller det interaktiva klientläget. Här är ett kodexempel med SDK:

<details>
<summary>Python Exempel</summary>

```python
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

async def run_product_search():
    server_params = StdioServerParameters(
        command="python",
        args=["server.py"],
    )
    async with stdio_client(server_params) as (reader, writer):
        async with ClientSession(reader, writer) as session:
            await session.initialize()
            result = await session.call_tool("product_search", arguments={"query": "best AI gadgets 2025"})
            print(result)
```
</details>

Alternativt, i interaktivt läge, välj `product_search` from the menu and enter your query when prompted.

**Parameters:**
- `query` (string): Produktfrågan

**Exempel på förfrågan:**

```json
{
  "query": "best AI gadgets 2025"
}
```

### qna

Hämtar direkta svar på frågor från sökmotorer.

**Så här anropar du verktyget:**

Du kan anropa `qna` från ditt eget script med MCP Python SDK, eller interaktivt med Inspector eller det interaktiva klientläget. Här är ett kodexempel med SDK:

<details>
<summary>Python Exempel</summary>

```python
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

async def run_qna():
    server_params = StdioServerParameters(
        command="python",
        args=["server.py"],
    )
    async with stdio_client(server_params) as (reader, writer):
        async with ClientSession(reader, writer) as session:
            await session.initialize()
            result = await session.call_tool("qna", arguments={"question": "what is artificial intelligence"})
            print(result)
```
</details>

Alternativt, i interaktivt läge, välj `qna` from the menu and enter your question when prompted.

**Parameters:**
- `question` (string): Frågan som ska besvaras

**Exempel på förfrågan:**

```json
{
  "question": "what is artificial intelligence"
}
```

## Koddetaljer

Denna sektion innehåller kodexempel och referenser för server- och klientimplementationerna.

<details>
<summary>Python</summary>

Se [`server.py`](../../../../05-AdvancedTopics/web-search-mcp/server.py) and [`client.py`](../../../../05-AdvancedTopics/web-search-mcp/client.py) för fullständiga implementationsdetaljer.

```python
# Example snippet from server.py:
import os
import httpx
# ...existing code...
```
</details>

## Avancerade koncept i denna lektion

Innan du börjar bygga, här är några viktiga avancerade koncept som kommer dyka upp genom kapitlet. Att förstå dessa hjälper dig att följa med, även om de är nya för dig:

- **Orkestrering av flera verktyg**: Detta innebär att flera olika verktyg (som webbsök, nyhetssök, produktsök och Q&A) körs inom en enda MCP-server. Det låter din server hantera många olika uppgifter, inte bara en.
- **Hantering av API-begränsningar**: Många externa API:er (som SerpAPI) begränsar hur många förfrågningar du kan göra under en viss tid. Bra kod kontrollerar dessa begränsningar och hanterar dem smidigt så att appen inte kraschar om du når gränsen.
- **Tolkning av strukturerad data**: API-svar är ofta komplexa och nästlade. Detta handlar om att omvandla dessa svar till rena, lättanvända format som är anpassade för LLM eller andra program.
- **Felåterhämtning**: Ibland går något fel – kanske nätverket brister eller API:et ger oväntade svar. Felåterhämtning betyder att din kod kan hantera dessa problem och ändå ge användbar feedback, istället för att krascha.
- **Parameterkontroll**: Det handlar om att kontrollera att alla indata till dina verktyg är korrekta och säkra att använda. Det inkluderar att sätta standardvärden och se till att typerna är rätt, vilket hjälper till att undvika buggar och förvirring.

Denna sektion hjälper dig att diagnostisera och lösa vanliga problem som kan uppstå när du arbetar med Web Search MCP Server. Om du stöter på fel eller oväntat beteende när du arbetar med servern, ger denna felsökningssektion lösningar på de vanligaste problemen. Läs igenom dessa tips innan du söker ytterligare hjälp – de löser ofta problem snabbt.

## Felsökning

När du arbetar med Web Search MCP Server kan du ibland stöta på problem – det är normalt när man utvecklar med externa API:er och nya verktyg. Denna sektion ger praktiska lösningar på de vanligaste problemen så att du snabbt kan komma på rätt spår igen. Om du får ett fel, börja här: tipsen nedan tar upp de problem som flest användare stöter på och kan ofta lösa ditt problem utan extra hjälp.

### Vanliga problem

Nedan är några av de vanligaste problemen användare stöter på, med tydliga förklaringar och steg för att lösa dem:

1. **Saknad SERPAPI_KEY i .env-filen**
   - Om du får felet `SERPAPI_KEY environment variable not found`, it means your application can't find the API key needed to access SerpAPI. To fix this, create a file named `.env` in your project root (if it doesn't already exist) and add a line like `SERPAPI_KEY=your_serpapi_key_here`. Make sure to replace `your_serpapi_key_here` with your actual key from the SerpAPI website.

2. **Module not found errors**
   - Errors such as `ModuleNotFoundError: No module named 'httpx'` indicate that a required Python package is missing. This usually happens if you haven't installed all the dependencies. To resolve this, run `pip install -r requirements.txt` in your terminal to install everything your project needs.

3. **Connection issues**
   - If you get an error like `Error during client execution`, it often means the client can't connect to the server, or the server isn't running as expected. Double-check that both the client and server are compatible versions, and that `server.py` is present and running in the correct directory. Restarting both the server and client can also help.

4. **SerpAPI errors**
   - Seeing `Search API returned error status: 401` means your SerpAPI key is missing, incorrect, or expired. Go to your SerpAPI dashboard, verify your key, and update your `.env`-filen vid behov. Om din nyckel är korrekt men du fortfarande får detta fel, kontrollera om din gratisnivå har slut på kvot.

### Debug-läge

Som standard loggar appen bara viktig information. Om du vill se mer detaljer om vad som händer (till exempel för att felsöka svåra problem) kan du aktivera DEBUG-läget. Det visar mycket mer om varje steg appen tar.

**Exempel: Normalt utdata**
```plaintext
2025-06-01 10:15:23,456 - __main__ - INFO - Calling general_search with params: {'query': 'open source LLMs'}
2025-06-01 10:15:24,123 - __main__ - INFO - Successfully called general_search

GENERAL_SEARCH RESULTS:
... (search results here) ...
```

**Exempel: DEBUG-utdata**
```plaintext
2025-06-01 10:15:23,456 - __main__ - INFO - Calling general_search with params: {'query': 'open source LLMs'}
2025-06-01 10:15:23,457 - httpx - DEBUG - HTTP Request: GET https://serpapi.com/search ...
2025-06-01 10:15:23,458 - httpx - DEBUG - HTTP Response: 200 OK ...
2025-06-01 10:15:24,123 - __main__ - INFO - Successfully called general_search

GENERAL_SEARCH RESULTS:
... (search results here) ...
```

Notera hur DEBUG-läget inkluderar extra rader om HTTP-förfrågningar, svar och andra interna detaljer. Detta kan vara mycket hjälpsamt vid felsökning.

För att aktivera DEBUG-läge, sätt loggningsnivån till DEBUG högst upp i din `client.py` or `server.py`:

<details>
<summary>Python</summary>

```python
# At the top of your client.py or server.py
import logging
logging.basicConfig(
    level=logging.DEBUG,  # Change from INFO to DEBUG
    format="%(asctime)s - %(name)s - %(levelname)s - %(message)s"
)
```
</details>

---

## Vad händer härnäst

- [5.10 Real Time Streaming](../mcp-realtimestreaming/README.md)

**Ansvarsfriskrivning**:  
Detta dokument har översatts med hjälp av AI-översättningstjänsten [Co-op Translator](https://github.com/Azure/co-op-translator). Även om vi strävar efter noggrannhet, vänligen var medveten om att automatiska översättningar kan innehålla fel eller brister. Det ursprungliga dokumentet på dess modersmål bör betraktas som den auktoritativa källan. För viktig information rekommenderas professionell mänsklig översättning. Vi ansvarar inte för eventuella missförstånd eller feltolkningar som uppstår vid användning av denna översättning.