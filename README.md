# ofac-sdn-mirror

Miroir opérateur du fichier **OFAC SDN** officiel (`sdn.xml`, format legacy `<sdnList>`),
rafraîchi quotidiennement par GitHub Action depuis :

    https://sanctionslistservice.ofac.treas.gov/api/PublicationPreview/exports/SDN.XML

## Pourquoi ce miroir

L'endpoint officiel répond **302 vers une URL S3 pré-signée** (valable 1 h, signature
différente à chaque requête). Les HTTPS Outcalls de l'Internet Computer ne suivent pas
les redirections, et une URL différente par nœud du subnet casserait le consensus.
`raw.githubusercontent.com` sert ce fichier **en 206 (Range) direct, sans redirection** —
consommable par un canister en téléchargement chunké.

URL consommée par `sm_list_ingest` (icp-EcoS / SanctionsMesh) :

    https://raw.githubusercontent.com/ilkinsem/ofac-sdn-mirror/main/sdn.xml

## Garanties

- Le workflow vérifie avant publication : format legacy `<sdnList>`, présence du
  garde-fou `Record_Count`, taille > 10 Mo (anti-troncature).
- `sdn.xml.sha256` publie l'empreinte du fichier courant — comparable au
  `sha256_hex` exposé par le canister (`list_status`) et au canary de divergence
  de SanctionsMesh (`report_observation`).
- `last_updated.txt` : horodatage UTC du dernier changement.
- L'historique est maintenu à un seul commit (amend + force push) pour contenir
  la taille du dépôt ; l'auditabilité passe par les sha256, pas par l'historique git.

## Chaîne de confiance

OFAC (source officielle) → cette Action (checks automatiques) → raw.githubusercontent
→ canister `sm_list_ingest` (re-vérifie taille annoncée == reçue, expose le sha256,
fail-closed `stale` en cas d'échec de refresh).
