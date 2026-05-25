<script>
  import Seo from "@shared/ui/seo.svelte";
  import { getSeoData } from "@shared/constants/seoData";
  import { setNavigationItems, clearNavigation } from "@shared/stores/navigation";
  import { onMount, onDestroy } from "svelte";
  import { router } from "tinro";
  import CodeBlock from "@shared/ui/codeBlock.svelte";
  import DockerIcon from "@shared/icons/dockerIcon.svelte";
  import { Terminal } from "lucide-svelte";

  const seo = getSeoData($router.path);

  onMount(() => {
    setNavigationItems(["Overview", "Versions", "Docker", "Parser", "Reverse Proxy", "Authentication", "Unraid", "That's It"], "/profilarr-setup/installation");
  });

  onDestroy(() => {
    clearNavigation();
  });

  const dockerComposeCode = `services:
  profilarr:
    image: ghcr.io/dictionarry-hub/profilarr:latest
    container_name: profilarr
    restart: unless-stopped
    ports:
      - "6868:6868"
    volumes:
      - ./config:/config
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK=022
      - TZ=Etc/UTC
      - PARSER_HOST=parser
      - PARSER_PORT=5000
    depends_on:
      parser:
        condition: service_healthy

  # Optional - only needed for CF/QP testing
  parser:
    image: ghcr.io/dictionarry-hub/profilarr-parser:latest
    container_name: profilarr-parser
    restart: unless-stopped
    expose:
      - "5000"`;

  const dockerCliCode = `docker run -d \\
  --name profilarr \\
  --restart unless-stopped \\
  -p 6868:6868 \\
  -v ./config:/config \\
  -e PUID=1000 \\
  -e PGID=1000 \\
  -e UMASK=022 \\
  -e TZ=Etc/UTC \\
  ghcr.io/dictionarry-hub/profilarr:latest`;

  const codeItems = [
    {
      title: "Docker Compose",
      code: dockerComposeCode,
      language: "yaml",
      icon: DockerIcon,
    },
    {
      title: "Docker CLI",
      code: dockerCliCode,
      language: "bash",
      icon: Terminal,
    },
  ];

  const originCode = `- ORIGIN=https://profilarr.your.domain`;

  let oidcActiveTab = 0;

  const oidcInlineCode = `services:
  profilarr:
    # ...
    environment:
      - AUTH=oidc
      - OIDC_DISCOVERY_URL=https://your-provider/.well-known/openid-configuration
      - OIDC_CLIENT_ID=your-client-id
      - OIDC_CLIENT_SECRET=your-client-secret`;

  const oidcSecretCode = `services:
  profilarr:
    # ...
    environment:
      - AUTH=oidc
      - OIDC_DISCOVERY_URL=https://your-provider/.well-known/openid-configuration
      - OIDC_CLIENT_ID=your-client-id
      - OIDC_CLIENT_SECRET_FILE=/run/secrets/oidc_client_secret
    secrets:
      - oidc_client_secret

secrets:
  oidc_client_secret:
    file: /path/to/your/secrets/oidc_client_secret`;

  const oidcItems = [
    { title: "Inline (Conventional)", code: oidcInlineCode, language: "yaml" },
    { title: "Docker Secret (Paranoid)", code: oidcSecretCode, language: "yaml" },
  ];

  const secretFileCreateCode = `printf '%s' 'your-client-secret-here' > /path/to/your/secrets/oidc_client_secret
chmod 600 /path/to/your/secrets/oidc_client_secret`;
</script>

<Seo title={seo.title} description={seo.description} image={seo.image} url={$router.path} />

<div>
  <h1 class="text-2xl font-bold text-neutral-900 dark:text-white mb-6">Installation</h1>

  <h2 class="text-xl font-semibold text-neutral-800 dark:text-neutral-200 mt-8 mb-4" id="overview">Overview</h2>
  <p class="text-neutral-700 dark:text-neutral-300 mt-6">
    Profilarr is currently only available through Docker, with native Windows support planned for the future.
  </p>

  <h2 class="text-xl font-semibold text-neutral-800 dark:text-neutral-200 mt-8 mb-4" id="versions">Versions</h2>
  <p class="text-neutral-700 dark:text-neutral-300 mt-6">
    Use <code class="bg-neutral-100 dark:bg-neutral-800 px-1.5 py-0.5 rounded">latest</code> unless you specifically want to test upcoming changes.
  </p>
  <p class="text-neutral-700 dark:text-neutral-300 mt-4">
    <code class="bg-neutral-100 dark:bg-neutral-800 px-1.5 py-0.5 rounded">latest</code> is the stable release image. <code class="bg-neutral-100 dark:bg-neutral-800 px-1.5 py-0.5 rounded">develop</code> is built from the testing branch and may include unfinished or untested changes. Tagged images, such as <code class="bg-neutral-100 dark:bg-neutral-800 px-1.5 py-0.5 rounded">v2.3.0</code>, can be used if you want to pin to a specific release.
  </p>

  <h2 class="text-xl font-semibold text-neutral-800 dark:text-neutral-200 mt-8 mb-4" id="docker">Docker</h2>
  <p class="text-neutral-700 dark:text-neutral-300 mt-6">
    To get started with Profilarr, you can use either Docker Compose or the Docker CLI. Choose the method that best fits
    your setup:
  </p>

  <div class="mt-4">
    <CodeBlock items={codeItems} />
  </div>

  <h2 class="text-xl font-semibold text-neutral-800 dark:text-neutral-200 mt-8 mb-4" id="parser">Parser</h2>
  <p class="text-neutral-700 dark:text-neutral-300 mt-6">
    The Docker Compose example includes a second container called <code class="bg-neutral-100 dark:bg-neutral-800 px-1.5 py-0.5 rounded">parser</code>. This service powers custom format and quality profile testing by using the same parsing logic as Radarr and Sonarr.
  </p>
  <p class="text-neutral-700 dark:text-neutral-300 mt-4">
    It is optional. Linking databases, syncing their configs, and running upgrades all work without it. It's primarily for database developers to test their configs. If you do not need testing, remove the <code class="bg-neutral-100 dark:bg-neutral-800 px-1.5 py-0.5 rounded">parser</code> service, the <code class="bg-neutral-100 dark:bg-neutral-800 px-1.5 py-0.5 rounded">PARSER_HOST</code>/<code class="bg-neutral-100 dark:bg-neutral-800 px-1.5 py-0.5 rounded">PARSER_PORT</code> environment variables, and the <code class="bg-neutral-100 dark:bg-neutral-800 px-1.5 py-0.5 rounded">depends_on</code> block.
  </p>

  <h2 class="text-xl font-semibold text-neutral-800 dark:text-neutral-200 mt-8 mb-4" id="reverse-proxy">Reverse Proxy</h2>
  <p class="text-neutral-700 dark:text-neutral-300 mt-6">
    If you are running Profilarr behind a reverse proxy (Traefik, Nginx Proxy Manager, Caddy, etc.), set the <code class="bg-neutral-100 dark:bg-neutral-800 px-1.5 py-0.5 rounded">ORIGIN</code> environment variable to the public URL Profilarr will be served from. Without it, Profilarr will not function correctly.
  </p>
  <p class="text-neutral-700 dark:text-neutral-300 mt-4">
    Add the following to the <code class="bg-neutral-100 dark:bg-neutral-800 px-1.5 py-0.5 rounded">environment</code> block of the <code class="bg-neutral-100 dark:bg-neutral-800 px-1.5 py-0.5 rounded">profilarr</code> service:
  </p>

  <div class="mt-4">
    <CodeBlock items={[{ title: "Environment", code: originCode, language: "yaml" }]} />
  </div>

  <h2 class="text-xl font-semibold text-neutral-800 dark:text-neutral-200 mt-8 mb-4" id="authentication">Authentication</h2>
  <p class="text-neutral-700 dark:text-neutral-300 mt-6">
    Profilarr supports three authentication modes, selected via the <code class="bg-neutral-100 dark:bg-neutral-800 px-1.5 py-0.5 rounded">AUTH</code> environment variable:
  </p>
  <ul class="text-neutral-700 dark:text-neutral-300 mt-4 ml-6 list-disc space-y-1">
    <li><code class="bg-neutral-100 dark:bg-neutral-800 px-1.5 py-0.5 rounded">on</code> — default, local username/password login required</li>
    <li><code class="bg-neutral-100 dark:bg-neutral-800 px-1.5 py-0.5 rounded">off</code> — no authentication (not recommended, use at your own risk)</li>
    <li><code class="bg-neutral-100 dark:bg-neutral-800 px-1.5 py-0.5 rounded">oidc</code> — single sign-on via an external identity provider</li>
  </ul>
  <p class="text-neutral-700 dark:text-neutral-300 mt-4">
    For OIDC, set <code class="bg-neutral-100 dark:bg-neutral-800 px-1.5 py-0.5 rounded">AUTH=oidc</code> along with the discovery URL, client ID, and client secret from your identity provider (Authentik, Authelia, Keycloak, etc.). The client secret can be supplied directly or mounted from a file:
  </p>

  <div class="mt-4">
    <CodeBlock items={oidcItems} bind:activeTab={oidcActiveTab} />
  </div>

  {#if oidcActiveTab === 1}
    <p class="text-neutral-700 dark:text-neutral-300 mt-4">
      When using the <strong>Docker Secret</strong> approach, create the secret file with no trailing newline- <code class="bg-neutral-100 dark:bg-neutral-800 px-1.5 py-0.5 rounded">echo</code> adds one by default and will silently break authentication:
    </p>

    <div class="mt-4">
      <CodeBlock items={[{ title: "Shell", code: secretFileCreateCode, language: "bash" }]} />
    </div>

    <p class="text-sm text-amber-700 dark:text-amber-400 mt-2">
      Note: <code class="bg-neutral-100 dark:bg-neutral-800 px-1.5 py-0.5 rounded">_FILE</code> works for any environment variable. If both <code class="bg-neutral-100 dark:bg-neutral-800 px-1.5 py-0.5 rounded">VAR</code> and <code class="bg-neutral-100 dark:bg-neutral-800 px-1.5 py-0.5 rounded">VAR_FILE</code> are set, the file value takes precedence.
    </p>
  {/if}

  <h2 class="text-xl font-semibold text-neutral-800 dark:text-neutral-200 mt-8 mb-4" id="unraid">Unraid</h2>
  <p class="text-neutral-700 dark:text-neutral-300 mt-6">
      The easiest way to install Profilarr on Unraid is through the <strong>Community Applications</strong> plugin. Simply
      search for "Profilarr" and install the container.
  </p>
  <p class="text-sm text-amber-700 dark:text-amber-400 mt-2">
      Note: the v2 template is separate from the one used for v1. The old template has been removed completely from Community Applications.
  </p>

  <h2 class="text-xl font-semibold text-neutral-800 dark:text-neutral-200 mt-8 mb-4" id="thats-it">That's It</h2>
  <p class="text-neutral-700 dark:text-neutral-300 mt-6">
    Once Profilarr is running, open the web UI and use the in-app onboarding to link your first database, connect Radarr or Sonarr, and configure sync. You can find it under <strong>Settings &gt; Onboarding</strong>.
  </p>
</div>
