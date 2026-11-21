# JCVERSA NoVaY — Premium Hybrid Anime Catalog Redesign

Redesign the NoVaY catalog tab into a hybrid discovery and complete paginated browsing experience in Midnight Noir & Crimson, introducing multi-criteria bottom-sheet filtering and richer real-data poster cards while preserving every existing search behavior, data integration, and animation.

## User Review & Critical Decisions

> [!IMPORTANT]
> All four architectural and UX choices below were confirmed during interactive clarification and govern this plan.

- **Confirmed Decision 1 — Hybrid Screen Layout**: Place the curated discovery sections (`À la une` spotlight hero, catalog selection, latest episodes, and random shuffle) directly above the complete, infinitely paginated responsive anime grid in one unified scrollable surface.
- **Confirmed Decision 2 — Honest Real-Data Filtering**: Expose multi-genre selection, `VF` / `VOSTFR` language filtering, format/type filtering, and sorting (`Par défaut`, `Récents`, `A–Z`, `Z–A`, plus year-based sorting when available), while dynamically surfacing year and status filters only from real metadata present in fetched catalog or search HTML. Unsupported concepts (`Highest Rated` scores, broadcast `Winter/Spring/Summer/Fall` seasons) are omitted rather than fabricated.
- **Confirmed Decision 3 — Filter Entry & Active State**: Place a dedicated filter button with an active-filter counter badge beside the search bar, open an advanced `ModalBottomSheet` with draft `Réinitialiser` and `Appliquer` controls, and display active criteria as removable chips directly below the search bar.
- **Confirmed Decision 4 — Poster Card Design**: Keep undistorted `2:3` poster proportions with a readable 2-line title below the poster, adding subtle corner pills for real language (`VF`/`VOSTFR`), library/offline status, and compact metadata only when present.

---

## 1. Overview & Core Concept

- **What It Does**: Evolves the catalog screen from an idle-spotlight-plus-search view into a full hybrid catalog where users can explore curated discovery rails, browse the entire paginated anime directory in an adaptive grid, and narrow or sort results using combined filters (`Genres`, `Langue`, `Type`, `Année`/`Statut` when exposed, and `Tri`) alongside instant multi-source search suggestions.
- **Target Audience / Persona**: Anime viewers on phones, foldables, and tablets browsing for new series by mood, language (`VF` vs. `VOSTFR`), format, or title without losing instant search speed.
- **Key Value**: Eliminates dead-end browsing—users immediately see what is featured and newly updated, can scroll seamlessly into the complete paginated catalog, and can combine multiple real filters without leaving the screen or hitting empty mock data.

---

## 2. User Experience & Visual Design

### Key User Flows

1. **Idle Hybrid Discovery & Complete Grid**:
   - At the top sits the collapsing `Catalogue` app bar, search field, and filter trigger button (showing a crimson count badge when `> 0` filters or non-default sort are active), followed by quick genre chips (or removable active filter chips when advanced filters are active).
   - Scrolling down reveals:
     1. **Featured Hero (`À la une`)** with parallax blurred backdrop, crisp `2:3` poster, genre/season subtitle, and primary `"Ouvrir la fiche"` CTA.
     2. **Discovery Rails** deduplicated cleanly against the hero: `"Sélection du catalogue"`, `"Derniers épisodes"` (with `VF`/`VOSTFR` pill and time label), `"Populaires"` / `"Tendances"` (distinct curated entries from catalog home), and `"Aléatoire"` (with live shuffle button).
     3. **Complete Catalog Grid (`Catalogue complet`)**: Header showing total loaded/matching count and active sort indicator, flowing directly into a responsive `2:3` poster grid with automatic infinite-scroll pagination and an unobtrusive bottom spinner/retry footer.
2. **Instant Search & Suggestions**:
   - Focusing or typing in the search field transitions smoothly into the multi-source suggestions view (persistent recent searches, library & history quick-jump cards, popular discovery tags, and 280ms debounced live catalog results with highlighted query spans).
   - Submitting a search or tapping a query suggestion transitions to the filtered result grid, where active bottom-sheet filters (genres, language, format, year/status, sort) also apply cleanly to search results.
3. **Advanced Filtering & Sorting Bottom Sheet**:
   - Tapping the filter button opens a `ModalBottomSheet` displaying a live preview count of matching entries, grouped filter sections (`Genres` multi-select wrap, `Langue`, `Format`, optional `Année` & `Statut` chips populated solely from real parsed entries, and `Tri`), plus `Réinitialiser` and `Appliquer` buttons.
   - Applying filters preserves state across detail navigation, shows removable chips under the search bar (e.g., `Action ×`, `VF ×`, `Film ×`, `Tri : A–Z ×`), and triggers paginated/filtered catalog loading or local refinement seamlessly.

### Visual Identity & Theme

- **Aesthetic Direction**: Midnight Noir & Crimson editorial streaming aesthetic—deep charcoal obsidian surfaces (`background`, `surfaceContainerLow`, `surfaceContainer`), crisp `0.5.dp` glass borders, and crimson focal accents (`primary`, `secondaryContainer`, `crimsonOnContainer`) with zero unnecessary outer boxes around cards.
- **Typography & Hierarchy**: Display and section headers use the existing NoVaY type scale (`headlineMedium` for hero title, `header` for section dividers with subtle leading accents, `titleSmall` 2-line capped titles under `2:3` posters, `labelSmall` micro-badges).
- **Card & Spacing Rhythm**: Adaptive `GridCells.Adaptive(140.dp)` with `12.dp` horizontal / `16.dp` vertical gutters, `16.dp` screen-edge margins, `>= 48.dp` touch targets on every chip, icon button, and action control, and edge-to-edge `LocalBottomChromePadding` clearance above the floating navigation dock.

### Interactive Feedback & Animation Preservation

Every existing animation and transition in the catalog and app shell is preserved intact and remains gated by `LocalSlateMotion.current.reduceMotion`:

- **Preserved App Bar & Screen Transitions**: `TopAppBarDefaults.enterAlwaysScrollBehavior`, `AnimatedContent` `motion.stateCrossfade()` (`210ms` EaseOutExpo fade+scale in / `130ms` FastOutExit fade out) between browse states, `novaySharedPoster` shared-element poster transitions to `ShowScreen`, and Predictive Back / rank-based screen transitions.
- **Preserved Hero & Card Micro-Interactions**: `SpotlightHeroCard` scroll parallax (`0.24f` via `derivedStateOf` + `graphicsLayer`), API 31+ `18.dp` blur backdrop (`0.32f` alpha fallback `0.16f`), `0.985f` press scale, `AnimeComfortableGridItem` `0.965f` press / `1.015f` hover spring scale (`effectFloatSpec()`), Material `ripple()`, and `LazyItemScope.animateItem()` on grid and rail items.
- **Preserved Loading & Chip Motion**: `ShimmerBox` / `PosterGridSkeleton` (`1250ms` draw-phase sweep), `GenreFilterChipBar` (`180ms` color animations), and `PosterImage` `160ms` Coil crossfade.
- **Complementary Additions**: Standard `ModalBottomSheet` slide/scrim transitions for the filter panel, `AnimatedVisibility` / `animateContentSize` (`sizeTransitionSpec`) for the removable active-filter chip strip and filter count badge, all disabled automatically when `reduceMotion` is true.

---

## 3. Key Product Decisions & Trade-Offs

- **Decision 1 — Unified `LazyVerticalGrid` with Full-Width `GridItemSpan` Discovery Header**:
  - *Chosen Approach*: Render the hero, discovery rails, active filter strip, and complete paginated grid inside a single `LazyVerticalGrid` using `span = { GridItemSpan(maxLineSpan) }` for horizontal discovery blocks and single-column spans for grid posters.
  - *Why*: Avoids nested vertical scrolling conflicts, preserves smooth `derivedStateOf` scroll-linked parallax on the hero, keeps infinite-scroll pagination detection (`firstVisibleItemIndex` / total count) deterministic, and scales naturally across portrait phones and landscape tablets.
  - *Alternatives Considered*: Separate `LazyColumn` + fixed-height grid (causes measurement overhead and clunky double scrolling).
- **Decision 2 — Real-Data-Only Multi-Criteria Filtering & Honest Field Availability**:
  - *Chosen Approach*: Parse real card/search metadata (`genres`, `language`, `format`/`type`, and `releaseYear` / `status` where anime-sama HTML provides them), map server-supported directory parameters (`page`, `type[]`, `genre[]`, `langue[]`) for paginated directory queries, and apply deterministic multi-criteria filtering + sorting across loaded/search datasets.
  - *Why*: Guarantees zero invented ratings, zero guessed broadcast seasons, and zero misleading empty states. When year or status attributes are absent from a provider response, those filter rows simply stay hidden rather than displaying empty or fake choices.
- **Decision 3 — Pagination & Memory Efficiency**:
  - *Chosen Approach*: Load the complete catalog page-by-page (`page = 1, 2, ...`) on demand with LRU TTL caching (`10 min`) in the data source, deduplicating by canonical catalog URL, preserving Coil memory/disk cache keys on `PosterImage`, and retaining filter/scroll state across `ShowScreen` navigation.

---

## 4. Technical Architecture & Data Strategy

```
┌──────────────────────────────────────────────────────────────────────┐
│                       Catalog UI Shell                               │
│  ┌──────────────────────┐   ┌───────────────────────┐                │
│  │ Search Bar + Filter  │──▶│ Active Removable Chips│                │
│  │ Trigger (Badge N)    │   │ & Genre Quick Bar     │                │
│  └──────────┬───────────┘   └──────────┬────────────┘                │
│             ▼                          ▼                             │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │         AnimatedContent (BrowseSearchDisplay State)            │  │
│  │  • HybridBrowseView (Unified LazyVerticalGrid):                │  │
│  │      Hero Spotlight -> Discovery Rails -> Paginated Grid       │  │
│  │  • Filtered / Search Results Grid (with count + pagination)    │  │
│  │  • Multi-Source Suggestions / Shimmer / Error / Empty Views    │  │
│  └──────────────────────────┬─────────────────────────────────────┘  │
└─────────────────────────────┼────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────────┐
│                 BrowseSearchController & Filter Engine               │
│  • CatalogFilterState (genres, languages, formats, years, statuses,  │
│    sortMode) + draft/apply/reset + activeFilterCount + removableChips│
│  • CatalogPaginationState (items, page, hasMore, isLoadingMore, err) │
│  • Pure Kotlin Filter & Sort Pipeline (multi-criteria, search-aware) │
└─────────────────────────────┬────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────────┐
│                   AnimeSource & Sama HTML Parsers                    │
│  • home() -> CatalogHome (heroes + recent episode cards)             │
│  • catalogPage(page, filter) -> Paginated directory + total/hasMore  │
│  • search(query) -> Live fetch.php results w/ parsed meta badges     │
│  • Card parser extracts real genres, format, year, language, status  │
└──────────────────────────────────────────────────────────────────────┘
```

### Data Model & State

- **`AnimeSearchResult` Extensions**: Optional nullable fields for real `format` (`Anime`, `Film`, `OAV`, `Scans`), `releaseYear`, and `status` parsed cleanly from search/catalog HTML without breaking existing callers or constructors.
- **`CatalogFilterState` & `CatalogSortOption`**: Immutable filter contract tracking `selectedGenres: Set<String>`, `selectedLanguage: String?`, `selectedFormat: String?`, `selectedYear: String?`, `selectedStatus: String?`, and `sort: CatalogSortOption` (`DEFAULT`, `RECENT`, `TITLE_ASC`, `TITLE_DESC`, `YEAR_DESC`), exposing `activeFilterCount`, `isDefault`, and `toRemovableChips()`.
- **`CatalogPage` & `CatalogPaginationState`**: Tracks `items: List<AnimeSearchResult>`, `page: Int`, `hasNextPage: Boolean`, `isLoadingNext: Boolean`, and `paginationError: String?`.

### Interactive Component & State Mapping

- **Search Bar & Filter Button**: Typing updates `queryField` and triggers 280ms debounced live lookup; tapping the filter icon opens `CatalogFilterBottomSheet` initialized with a draft copy of `CatalogFilterState`.
- **Filter Bottom Sheet**: Multi-select genre chips, single-select language/format/year/status/sort pills dynamically compute matching counts; `Réinitialiser` clears draft filters while `Appliquer` commits `CatalogFilterState`, synchronizes the quick genre bar, and either filters in memory or fetches matching paginated directory results.
- **Removable Active Filter Chips**: Each active filter chip invokes a targeted removal action (`removeGenre`, `clearLanguage`, `clearFormat`, `clearYear`, `clearStatus`, `resetSort`) and refreshes the active catalog/search view immediately.
- **Hybrid Grid & Infinite Scroll**: Scrolling within 6 items of the grid tail triggers `loadNextCatalogPage()` while deduplicating URLs and exposing a clean inline footer retry button if a page fetch fails.
