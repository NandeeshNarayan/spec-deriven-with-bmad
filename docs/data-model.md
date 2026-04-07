# LIPILY — Complete Data Model Reference

> **Purpose:** Single source of truth for every database table, column, index, constraint, and RLS policy.  
> **Agent rule:** When implementing any feature, check this file first. Never invent table or column names.  
> **Stack:** Supabase PostgreSQL, Supavisor pooler port 6543 only.  
> **Last updated:** 2026-04-07

---

## Global Conventions

| Convention | Value |
|---|---|
| Primary keys | `UUID DEFAULT gen_random_uuid()` |
| Timestamps | `TIMESTAMPTZ NOT NULL DEFAULT NOW()` |
| Soft deletes | `deleted_at TIMESTAMPTZ` (NULL = active) |
| String IDs | Never — always UUID |
| Booleans | `BOOLEAN NOT NULL DEFAULT FALSE` |
| All money | `INT` cents (never FLOAT) |
| RLS | Enabled on every table; deny-all default |
| Content source | `ContentSource ENUM` — see below |

```sql
CREATE TYPE content_source AS ENUM (
  'ai_generated',
  'human_confirmed',
  'human_edited',
  'human_written'
);
```

---

## Table Reference

### `profiles`
Extends `auth.users` (Supabase). Created by `on_auth_user_created` trigger.

```sql
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  email TEXT NOT NULL,
  display_name TEXT,
  username TEXT UNIQUE,
  bio TEXT CHECK (char_length(bio) <= 160),
  writer_type TEXT, -- 'student' | 'emerging' | 'professional' | 'hobbyist'
  writing_goal TEXT,
  preferred_format TEXT,
  avatar_url TEXT,
  ui_language TEXT NOT NULL DEFAULT 'en',
  is_admin BOOLEAN NOT NULL DEFAULT FALSE,
  is_suspended BOOLEAN NOT NULL DEFAULT FALSE,
  is_banned BOOLEAN NOT NULL DEFAULT FALSE,
  warning_count INT NOT NULL DEFAULT 0,
  follower_count INT NOT NULL DEFAULT 0,
  following_count INT NOT NULL DEFAULT 0,
  onboarding_completed BOOLEAN NOT NULL DEFAULT FALSE,
  producer_discoverable BOOLEAN NOT NULL DEFAULT TRUE,
  default_contact_info JSONB,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
CREATE POLICY "profiles_select_own" ON profiles FOR SELECT USING (auth.uid() = id);
CREATE POLICY "profiles_update_own" ON profiles FOR UPDATE USING (auth.uid() = id);
CREATE POLICY "profiles_select_public" ON profiles FOR SELECT USING (true); -- public read
```

---

### `scripts`

```sql
CREATE TABLE scripts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  title TEXT NOT NULL DEFAULT 'Untitled Script',
  format TEXT NOT NULL DEFAULT 'feature', -- 'feature' | 'pilot' | 'short' | 'episode' | 'web_series'
  language TEXT NOT NULL DEFAULT 'en',
  page_count INT NOT NULL DEFAULT 0,
  word_count INT NOT NULL DEFAULT 0,
  y_doc_state BYTEA, -- Yjs encoded document state
  is_published BOOLEAN NOT NULL DEFAULT FALSE,
  published_at TIMESTAMPTZ,
  title_page_data JSONB,
  pitch_package JSONB,
  formatting_preferences JSONB,
  writers_room_enabled BOOLEAN NOT NULL DEFAULT FALSE,
  is_template BOOLEAN NOT NULL DEFAULT FALSE,
  deleted_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_scripts_user ON scripts(user_id, deleted_at);
CREATE INDEX idx_scripts_published ON scripts(is_published, published_at DESC);

ALTER TABLE scripts ENABLE ROW LEVEL SECURITY;
CREATE POLICY "scripts_owner" ON scripts FOR ALL USING (auth.uid() = user_id);
CREATE POLICY "scripts_collaborator" ON scripts FOR SELECT
  USING (EXISTS (SELECT 1 FROM collaborators WHERE script_id = scripts.id AND user_id = auth.uid()));
```

---

### `blocks`

```sql
CREATE TABLE blocks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  script_id UUID NOT NULL REFERENCES scripts(id) ON DELETE CASCADE,
  block_type TEXT NOT NULL,
  -- block_type values: 'scene_heading' | 'action' | 'character' | 'dialogue'
  --                   | 'parenthetical' | 'transition' | 'general'
  content TEXT NOT NULL DEFAULT '',
  position BIGINT NOT NULL, -- gap strategy: 1000, 2000, 3000…
  content_source content_source NOT NULL DEFAULT 'human_written',
  is_active BOOLEAN NOT NULL DEFAULT TRUE, -- false for inactive dialogue variants
  parent_block_id UUID REFERENCES blocks(id),
  variant_index INT NOT NULL DEFAULT 0,
  is_locked BOOLEAN NOT NULL DEFAULT FALSE,
  revision_mark BOOLEAN NOT NULL DEFAULT FALSE,
  revision_colour TEXT,
  deleted_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_blocks_script ON blocks(script_id, position) WHERE deleted_at IS NULL;
CREATE INDEX idx_blocks_variants ON blocks(parent_block_id) WHERE parent_block_id IS NOT NULL;

ALTER TABLE blocks ENABLE ROW LEVEL SECURITY;
CREATE POLICY "blocks_script_access" ON blocks FOR ALL
  USING (EXISTS (
    SELECT 1 FROM scripts s
    LEFT JOIN collaborators c ON c.script_id = s.id AND c.user_id = auth.uid()
    WHERE s.id = blocks.script_id AND (s.user_id = auth.uid() OR c.user_id IS NOT NULL)
  ));
```

---

### `collaborators`

```sql
CREATE TABLE collaborators (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  script_id UUID NOT NULL REFERENCES scripts(id) ON DELETE CASCADE,
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  invited_email TEXT, -- null once user joins
  role TEXT NOT NULL DEFAULT 'editor', -- 'owner' | 'showrunner' | 'editor' | 'commenter' | 'viewer'
  invite_token TEXT UNIQUE,
  invite_expires_at TIMESTAMPTZ,
  joined_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE(script_id, user_id)
);

ALTER TABLE collaborators ENABLE ROW LEVEL SECURITY;
CREATE POLICY "collaborators_owner_manage" ON collaborators FOR ALL
  USING (EXISTS (SELECT 1 FROM scripts WHERE id = script_id AND user_id = auth.uid()));
CREATE POLICY "collaborators_self_read" ON collaborators FOR SELECT
  USING (user_id = auth.uid());
```

---

### `drafts`

```sql
CREATE TABLE drafts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  script_id UUID NOT NULL REFERENCES scripts(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES profiles(id),
  name TEXT NOT NULL,
  draft_type TEXT NOT NULL DEFAULT 'working',
  -- 'working' | 'first' | 'revised' | 'production' | 'wga_registration'
  y_doc_snapshot BYTEA NOT NULL,
  page_count INT NOT NULL DEFAULT 0,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_drafts_script ON drafts(script_id, created_at DESC);

ALTER TABLE drafts ENABLE ROW LEVEL SECURITY;
CREATE POLICY "drafts_owner" ON drafts FOR ALL
  USING (EXISTS (SELECT 1 FROM scripts WHERE id = script_id AND user_id = auth.uid()));
```

---

### `script_branches`

```sql
CREATE TABLE script_branches (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  script_id UUID NOT NULL REFERENCES scripts(id) ON DELETE CASCADE,
  created_by UUID NOT NULL REFERENCES profiles(id),
  branch_name TEXT NOT NULL,
  y_doc_state BYTEA NOT NULL,
  status TEXT NOT NULL DEFAULT 'open', -- 'open' | 'merged' | 'rejected' | 'closed'
  description TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  merged_at TIMESTAMPTZ
);

ALTER TABLE script_branches ENABLE ROW LEVEL SECURITY;
CREATE POLICY "branches_collaborator" ON script_branches FOR ALL
  USING (EXISTS (
    SELECT 1 FROM collaborators WHERE script_id = script_branches.script_id AND user_id = auth.uid()
  ));
```

---

### `comments`

```sql
CREATE TABLE comments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  script_id UUID NOT NULL REFERENCES scripts(id) ON DELETE CASCADE,
  block_id UUID REFERENCES blocks(id) ON DELETE SET NULL,
  user_id UUID NOT NULL REFERENCES profiles(id),
  parent_comment_id UUID REFERENCES comments(id),
  content TEXT NOT NULL,
  text_anchor_start INT,
  text_anchor_end INT,
  is_resolved BOOLEAN NOT NULL DEFAULT FALSE,
  resolved_by UUID REFERENCES profiles(id),
  resolved_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_comments_script ON comments(script_id, is_resolved);
CREATE INDEX idx_comments_block ON comments(block_id) WHERE block_id IS NOT NULL;

ALTER TABLE comments ENABLE ROW LEVEL SECURITY;
CREATE POLICY "comments_collaborator" ON comments FOR ALL
  USING (EXISTS (
    SELECT 1 FROM collaborators WHERE script_id = comments.script_id AND user_id = auth.uid()
  ));
```

---

### `scene_intelligence`

```sql
CREATE TABLE scene_intelligence (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  script_id UUID NOT NULL REFERENCES scripts(id) ON DELETE CASCADE,
  block_id UUID NOT NULL REFERENCES blocks(id) ON DELETE CASCADE,
  pillar TEXT NOT NULL,
  -- 'story_structure' | 'character_psychology' | 'visual_grammar'
  -- | 'emotional_arc' | 'thematic_resonance'
  content TEXT NOT NULL,
  content_source content_source NOT NULL DEFAULT 'ai_generated',
  model_used TEXT NOT NULL,
  approved_at TIMESTAMPTZ,
  approved_by UUID REFERENCES profiles(id),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE(block_id, pillar)
);

ALTER TABLE scene_intelligence ENABLE ROW LEVEL SECURITY;
CREATE POLICY "scene_intel_owner" ON scene_intelligence FOR ALL
  USING (EXISTS (SELECT 1 FROM scripts WHERE id = script_id AND user_id = auth.uid()));
CREATE POLICY "scene_intel_collaborator" ON scene_intelligence FOR SELECT
  USING (EXISTS (SELECT 1 FROM collaborators WHERE script_id = scene_intelligence.script_id AND user_id = auth.uid()));
```

---

### `story_bible`

```sql
CREATE TABLE story_bible (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  script_id UUID NOT NULL REFERENCES scripts(id) ON DELETE CASCADE,
  category TEXT NOT NULL, -- 'character' | 'location' | 'theme' | 'premise'
  title TEXT NOT NULL,
  content TEXT NOT NULL,
  content_source content_source NOT NULL DEFAULT 'human_written',
  sort_order INT NOT NULL DEFAULT 0,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_story_bible_script ON story_bible(script_id, category);

ALTER TABLE story_bible ENABLE ROW LEVEL SECURITY;
CREATE POLICY "story_bible_access" ON story_bible FOR ALL
  USING (EXISTS (
    SELECT 1 FROM scripts s
    LEFT JOIN collaborators c ON c.script_id = s.id AND c.user_id = auth.uid()
    WHERE s.id = story_bible.script_id AND (s.user_id = auth.uid() OR c.user_id IS NOT NULL)
  ));
```

---

### `agent_approvals`

```sql
CREATE TABLE agent_approvals (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  script_id UUID NOT NULL REFERENCES scripts(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES profiles(id),
  feature TEXT NOT NULL,
  -- 'scene_intelligence' | 'dialogue_suggest' | 'continue_scene'
  -- | 'logline_pitch' | 'voice_guardian' | 'continuity_checker'
  prompt_hash TEXT NOT NULL,
  model_used TEXT NOT NULL,
  input_tokens INT NOT NULL,
  output_tokens INT NOT NULL,
  accepted BOOLEAN,
  accepted_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_agent_approvals_user_date ON agent_approvals(user_id, created_at DESC);

ALTER TABLE agent_approvals ENABLE ROW LEVEL SECURITY;
CREATE POLICY "approvals_owner" ON agent_approvals FOR ALL USING (user_id = auth.uid());
```

---

### `continuity_flags`

```sql
CREATE TABLE continuity_flags (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  script_id UUID NOT NULL REFERENCES scripts(id) ON DELETE CASCADE,
  block_id UUID REFERENCES blocks(id) ON DELETE SET NULL,
  flag_type TEXT NOT NULL,
  -- 'timeline' | 'character' | 'object' | 'location' | 'physical' | 'voice'
  severity TEXT NOT NULL DEFAULT 'warning', -- 'critical' | 'warning' | 'note'
  title TEXT NOT NULL,
  description TEXT NOT NULL,
  is_dismissed BOOLEAN NOT NULL DEFAULT FALSE,
  dismissed_at TIMESTAMPTZ,
  suppress_until TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_continuity_flags_script ON continuity_flags(script_id, is_dismissed);

ALTER TABLE continuity_flags ENABLE ROW LEVEL SECURITY;
CREATE POLICY "flags_owner" ON continuity_flags FOR ALL
  USING (EXISTS (SELECT 1 FROM scripts WHERE id = script_id AND user_id = auth.uid()));
```

---

### `writing_sessions`

```sql
CREATE TABLE writing_sessions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES profiles(id),
  script_id UUID NOT NULL REFERENCES scripts(id) ON DELETE CASCADE,
  started_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  ended_at TIMESTAMPTZ,
  words_written INT NOT NULL DEFAULT 0,
  pages_written FLOAT NOT NULL DEFAULT 0
);

CREATE INDEX idx_writing_sessions_user ON writing_sessions(user_id, started_at DESC);

ALTER TABLE writing_sessions ENABLE ROW LEVEL SECURITY;
CREATE POLICY "sessions_owner" ON writing_sessions FOR ALL USING (user_id = auth.uid());
```

---

### `beat_board_cards`

```sql
CREATE TABLE beat_board_cards (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  script_id UUID NOT NULL REFERENCES scripts(id) ON DELETE CASCADE,
  title TEXT NOT NULL,
  description TEXT,
  beat_framework TEXT NOT NULL DEFAULT 'three_act',
  -- 'three_act' | 'save_the_cat'
  beat_position TEXT NOT NULL,
  -- Three-act: 'act1' | 'midpoint' | 'act2a' | 'act2b' | 'act3'
  -- Save the Cat: 'opening_image' | 'theme_stated' | 'setup' ... (15 positions)
  linked_scene_id UUID REFERENCES blocks(id),
  sort_order INT NOT NULL DEFAULT 0,
  is_ai_suggested BOOLEAN NOT NULL DEFAULT FALSE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

ALTER TABLE beat_board_cards ENABLE ROW LEVEL SECURITY;
CREATE POLICY "beat_board_owner" ON beat_board_cards FOR ALL
  USING (EXISTS (SELECT 1 FROM scripts WHERE id = script_id AND user_id = auth.uid()));
```

---

### `shots`

```sql
CREATE TABLE shots (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  script_id UUID NOT NULL REFERENCES scripts(id) ON DELETE CASCADE,
  scene_block_id UUID NOT NULL REFERENCES blocks(id) ON DELETE CASCADE,
  shot_index INT NOT NULL DEFAULT 0,
  prompt_used TEXT NOT NULL,
  s3_url TEXT NOT NULL,
  revised_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_shots_scene ON shots(scene_block_id);

ALTER TABLE shots ENABLE ROW LEVEL SECURITY;
CREATE POLICY "shots_owner" ON shots FOR ALL
  USING (EXISTS (SELECT 1 FROM scripts WHERE id = script_id AND user_id = auth.uid()));
```

---

### `inline_tags`

```sql
CREATE TABLE inline_tags (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  block_id UUID NOT NULL REFERENCES blocks(id) ON DELETE CASCADE,
  script_id UUID NOT NULL REFERENCES scripts(id) ON DELETE CASCADE,
  tag_type TEXT NOT NULL,
  -- 'prop' | 'sfx' | 'costume' | 'stunt' | 'location' | 'cast'
  -- | 'vfx' | 'animal' | 'vehicle' | 'music'
  tagged_text TEXT NOT NULL,
  char_start INT NOT NULL,
  char_end INT NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_inline_tags_script ON inline_tags(script_id, tag_type);

ALTER TABLE inline_tags ENABLE ROW LEVEL SECURITY;
CREATE POLICY "tags_owner" ON inline_tags FOR ALL
  USING (EXISTS (SELECT 1 FROM scripts WHERE id = script_id AND user_id = auth.uid()));
```

---

### `research_notes`

```sql
CREATE TABLE research_notes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  script_id UUID NOT NULL REFERENCES scripts(id) ON DELETE CASCADE,
  scene_block_id UUID REFERENCES blocks(id) ON DELETE SET NULL,
  user_id UUID NOT NULL REFERENCES profiles(id),
  title TEXT,
  content TEXT NOT NULL,
  source_url TEXT,
  og_title TEXT,
  og_image TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

ALTER TABLE research_notes ENABLE ROW LEVEL SECURITY;
CREATE POLICY "research_owner" ON research_notes FOR ALL USING (user_id = auth.uid());
```

---

### `formatting_preferences`

```sql
CREATE TABLE formatting_preferences (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  script_id UUID NOT NULL UNIQUE REFERENCES scripts(id) ON DELETE CASCADE,
  page_size TEXT NOT NULL DEFAULT 'us_letter', -- 'us_letter' | 'a4'
  font_family TEXT NOT NULL DEFAULT 'courier_prime',
  line_spacing FLOAT NOT NULL DEFAULT 1.0,
  margin_top FLOAT NOT NULL DEFAULT 1.0,
  margin_bottom FLOAT NOT NULL DEFAULT 1.0,
  margin_left FLOAT NOT NULL DEFAULT 1.5,
  margin_right FLOAT NOT NULL DEFAULT 1.0,
  smart_flow_mode TEXT NOT NULL DEFAULT 'full', -- 'full' | 'minimal'
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

ALTER TABLE formatting_preferences ENABLE ROW LEVEL SECURITY;
CREATE POLICY "formatting_owner" ON formatting_preferences FOR ALL
  USING (EXISTS (SELECT 1 FROM scripts WHERE id = script_id AND user_id = auth.uid()));
```

---

### `user_follows`

```sql
CREATE TABLE user_follows (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  follower_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  following_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE (follower_id, following_id)
);

CREATE INDEX idx_user_follows_follower ON user_follows(follower_id);
CREATE INDEX idx_user_follows_following ON user_follows(following_id);

ALTER TABLE user_follows ENABLE ROW LEVEL SECURITY;
CREATE POLICY "follows_own" ON user_follows FOR ALL USING (follower_id = auth.uid());
CREATE POLICY "follows_public_read" ON user_follows FOR SELECT USING (true);
```

---

### `username_change_log`

```sql
CREATE TABLE username_change_log (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES profiles(id),
  old_username TEXT NOT NULL,
  new_username TEXT NOT NULL,
  changed_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

ALTER TABLE username_change_log ENABLE ROW LEVEL SECURITY;
CREATE POLICY "username_log_owner" ON username_change_log FOR SELECT USING (user_id = auth.uid());
```

---

### `script_publications`

```sql
CREATE TABLE script_publications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  script_id UUID NOT NULL REFERENCES scripts(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES profiles(id),
  logline TEXT NOT NULL,
  genre TEXT[] NOT NULL,
  format TEXT NOT NULL,
  content_rating TEXT NOT NULL DEFAULT 'general',
  -- 'general' | 'teen' | 'mature'
  visibility TEXT NOT NULL DEFAULT 'public',
  -- 'public' | 'unlisted'
  cover_url TEXT,
  language TEXT NOT NULL DEFAULT 'en',
  like_count INT NOT NULL DEFAULT 0,
  bookmark_count INT NOT NULL DEFAULT 0,
  listen_play_count INT NOT NULL DEFAULT 0,
  review_count INT NOT NULL DEFAULT 0,
  published_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_script_publications_user ON script_publications(user_id);
CREATE INDEX idx_script_publications_published ON script_publications(published_at DESC);
CREATE INDEX idx_script_publications_visibility ON script_publications(visibility, content_rating);

ALTER TABLE script_publications ENABLE ROW LEVEL SECURITY;
CREATE POLICY "publications_owner" ON script_publications FOR ALL USING (user_id = auth.uid());
CREATE POLICY "publications_public_read" ON script_publications FOR SELECT
  USING (visibility IN ('public', 'unlisted'));
```

---

### `script_likes`

```sql
CREATE TABLE script_likes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  publication_id UUID NOT NULL REFERENCES script_publications(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE(user_id, publication_id)
);

ALTER TABLE script_likes ENABLE ROW LEVEL SECURITY;
CREATE POLICY "likes_own" ON script_likes FOR ALL USING (user_id = auth.uid());
CREATE POLICY "likes_count_read" ON script_likes FOR SELECT USING (true);
```

---

### `script_bookmarks`

```sql
CREATE TABLE script_bookmarks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  publication_id UUID NOT NULL REFERENCES script_publications(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE(user_id, publication_id)
);

ALTER TABLE script_bookmarks ENABLE ROW LEVEL SECURITY;
CREATE POLICY "bookmarks_own" ON script_bookmarks FOR ALL USING (user_id = auth.uid());
```

---

### `script_views`

```sql
CREATE TABLE script_views (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  publication_id UUID NOT NULL REFERENCES script_publications(id) ON DELETE CASCADE,
  viewer_id UUID REFERENCES profiles(id),
  viewed_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_script_views_publication ON script_views(publication_id, viewed_at DESC);

ALTER TABLE script_views ENABLE ROW LEVEL SECURITY;
CREATE POLICY "views_insert_any" ON script_views FOR INSERT WITH CHECK (true);
CREATE POLICY "views_owner_read" ON script_views FOR SELECT
  USING (EXISTS (SELECT 1 FROM script_publications WHERE id = publication_id AND user_id = auth.uid()));
```

---

### `leaderboard_snapshots`

```sql
CREATE TABLE leaderboard_snapshots (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  snapshot_type TEXT NOT NULL, -- 'weekly' | 'all_time'
  rank INT NOT NULL,
  publication_id UUID NOT NULL REFERENCES script_publications(id),
  score INT NOT NULL,
  view_count INT NOT NULL,
  like_count INT NOT NULL,
  listen_play_count INT NOT NULL,
  review_count INT NOT NULL,
  computed_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_leaderboard_snapshots_type_rank
  ON leaderboard_snapshots(snapshot_type, rank, computed_at DESC);

ALTER TABLE leaderboard_snapshots ENABLE ROW LEVEL SECURITY;
CREATE POLICY "leaderboard_public_read" ON leaderboard_snapshots FOR SELECT USING (true);
```

---

### `listen_productions`

```sql
CREATE TABLE listen_productions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  script_id UUID NOT NULL REFERENCES scripts(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES profiles(id),
  status TEXT NOT NULL DEFAULT 'queued',
  -- 'queued' | 'processing' | 'ready' | 'failed'
  current_step TEXT, -- e.g., 'generate_audio'
  timeline_manifest JSONB,
  duration_seconds INT,
  music_track_id TEXT,
  retry_count INT NOT NULL DEFAULT 0,
  error_message TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  completed_at TIMESTAMPTZ,
  UNIQUE(script_id)
);

ALTER TABLE listen_productions ENABLE ROW LEVEL SECURITY;
CREATE POLICY "listen_owner" ON listen_productions FOR ALL USING (user_id = auth.uid());
CREATE POLICY "listen_public_read" ON listen_productions FOR SELECT
  USING (EXISTS (SELECT 1 FROM script_publications WHERE script_id = listen_productions.script_id));
```

---

### `listen_voice_assignments`

```sql
CREATE TABLE listen_voice_assignments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  listen_id UUID NOT NULL REFERENCES listen_productions(id) ON DELETE CASCADE,
  character_name TEXT NOT NULL,
  voice_id TEXT NOT NULL,
  pitch FLOAT NOT NULL DEFAULT 1.0,
  pace FLOAT NOT NULL DEFAULT 1.0,
  accent_hint TEXT
);

ALTER TABLE listen_voice_assignments ENABLE ROW LEVEL SECURITY;
CREATE POLICY "voice_assignments_owner" ON listen_voice_assignments FOR ALL
  USING (EXISTS (SELECT 1 FROM listen_productions WHERE id = listen_id AND user_id = auth.uid()));
```

---

### `tts_voice_cache`

```sql
CREATE TABLE tts_voice_cache (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  script_id UUID NOT NULL REFERENCES scripts(id) ON DELETE CASCADE,
  character_name TEXT NOT NULL,
  voice_id TEXT NOT NULL,
  pitch FLOAT NOT NULL DEFAULT 1.0,
  pace FLOAT NOT NULL DEFAULT 1.0,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE(script_id, character_name)
);

ALTER TABLE tts_voice_cache ENABLE ROW LEVEL SECURITY;
CREATE POLICY "tts_cache_owner" ON tts_voice_cache FOR ALL
  USING (EXISTS (SELECT 1 FROM scripts WHERE id = script_id AND user_id = auth.uid()));
```

---

### `reviews`

```sql
CREATE TABLE reviews (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  script_id UUID NOT NULL REFERENCES scripts(id),
  publication_id UUID NOT NULL REFERENCES script_publications(id),
  writer_id UUID NOT NULL REFERENCES profiles(id),
  reviewer_id UUID NOT NULL REFERENCES profiles(id),
  status TEXT NOT NULL DEFAULT 'pending',
  -- 'pending' | 'in_progress' | 'completed' | 'overdue' | 'refunded' | 'declined'
  price_cents INT NOT NULL,
  stripe_payment_intent_id TEXT NOT NULL,
  stripe_refund_id TEXT,
  overall_rating INT CHECK (overall_rating >= 1 AND overall_rating <= 5),
  section_ratings JSONB, -- { concept: 4, structure: 3, character: 5, dialogue: 4, visuals: 3, market: 4 }
  section_text JSONB,
  draft_content JSONB,
  helpful_count INT NOT NULL DEFAULT 0,
  deadline_at TIMESTAMPTZ NOT NULL,
  completed_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_reviews_reviewer ON reviews(reviewer_id, status);
CREATE INDEX idx_reviews_writer ON reviews(writer_id);
CREATE INDEX idx_reviews_deadline ON reviews(deadline_at) WHERE status IN ('pending', 'in_progress');

ALTER TABLE reviews ENABLE ROW LEVEL SECURITY;
CREATE POLICY "reviews_writer" ON reviews FOR SELECT USING (writer_id = auth.uid());
CREATE POLICY "reviews_reviewer" ON reviews FOR ALL USING (reviewer_id = auth.uid());
```

---

### `reviewer_profiles`

```sql
CREATE TABLE reviewer_profiles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL UNIQUE REFERENCES profiles(id),
  status TEXT NOT NULL DEFAULT 'pending',
  -- 'pending' | 'approved' | 'rejected' | 'paused'
  bio TEXT NOT NULL,
  portfolio_url TEXT,
  sample_analyses JSONB,
  price_cents INT NOT NULL DEFAULT 5000,
  stripe_connect_account_id TEXT,
  stripe_connect_enabled BOOLEAN NOT NULL DEFAULT FALSE,
  visible_in_marketplace BOOLEAN NOT NULL DEFAULT FALSE,
  total_earned_cents BIGINT NOT NULL DEFAULT 0,
  reviews_completed INT NOT NULL DEFAULT 0,
  on_time_rate FLOAT NOT NULL DEFAULT 1.0,
  helpful_rate FLOAT NOT NULL DEFAULT 0.0,
  average_rating FLOAT,
  applied_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  approved_at TIMESTAMPTZ,
  paused_at TIMESTAMPTZ
);

ALTER TABLE reviewer_profiles ENABLE ROW LEVEL SECURITY;
CREATE POLICY "reviewer_profile_own" ON reviewer_profiles FOR ALL USING (user_id = auth.uid());
CREATE POLICY "reviewer_profile_public_read" ON reviewer_profiles FOR SELECT
  USING (visible_in_marketplace = TRUE);
```

---

### `payout_errors`

```sql
CREATE TABLE payout_errors (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  review_id UUID NOT NULL REFERENCES reviews(id),
  reviewer_id UUID NOT NULL REFERENCES profiles(id),
  amount_cents INT NOT NULL,
  error_message TEXT NOT NULL,
  resolved BOOLEAN NOT NULL DEFAULT FALSE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

ALTER TABLE payout_errors ENABLE ROW LEVEL SECURITY;
CREATE POLICY "payout_errors_deny_all" ON payout_errors FOR ALL USING (false);
-- only service role (Rust) can access
```

---

### `producer_profiles`

```sql
CREATE TABLE producer_profiles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL UNIQUE REFERENCES profiles(id),
  company_name TEXT NOT NULL,
  portfolio_url TEXT,
  statement TEXT NOT NULL,
  status TEXT NOT NULL DEFAULT 'pending',
  -- 'pending' | 'approved' | 'rejected'
  applied_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  approved_at TIMESTAMPTZ
);

ALTER TABLE producer_profiles ENABLE ROW LEVEL SECURITY;
CREATE POLICY "producer_profile_own" ON producer_profiles FOR ALL USING (user_id = auth.uid());
CREATE POLICY "producer_profile_admin_read" ON producer_profiles FOR SELECT
  USING (EXISTS (SELECT 1 FROM profiles WHERE id = auth.uid() AND is_admin = TRUE));
```

---

### `producer_saved_scripts`

```sql
CREATE TABLE producer_saved_scripts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  producer_id UUID NOT NULL REFERENCES producer_profiles(id),
  publication_id UUID NOT NULL REFERENCES script_publications(id),
  notes TEXT,
  saved_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE(producer_id, publication_id)
);

ALTER TABLE producer_saved_scripts ENABLE ROW LEVEL SECURITY;
CREATE POLICY "producer_saved_own" ON producer_saved_scripts FOR ALL
  USING (EXISTS (SELECT 1 FROM producer_profiles WHERE id = producer_id AND user_id = auth.uid()));
```

---

### `producer_contact_requests`

```sql
CREATE TABLE producer_contact_requests (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  producer_id UUID NOT NULL REFERENCES producer_profiles(id),
  writer_id UUID NOT NULL REFERENCES profiles(id),
  publication_id UUID NOT NULL REFERENCES script_publications(id),
  message TEXT NOT NULL,
  purpose TEXT NOT NULL,
  status TEXT NOT NULL DEFAULT 'pending',
  -- 'pending' | 'replied' | 'declined'
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

ALTER TABLE producer_contact_requests ENABLE ROW LEVEL SECURITY;
CREATE POLICY "contact_request_writer" ON producer_contact_requests FOR SELECT
  USING (writer_id = auth.uid());
CREATE POLICY "contact_request_producer" ON producer_contact_requests FOR ALL
  USING (EXISTS (SELECT 1 FROM producer_profiles WHERE id = producer_id AND user_id = auth.uid()));
```

---

### `subscriptions`

```sql
CREATE TABLE subscriptions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL UNIQUE REFERENCES profiles(id),
  plan TEXT NOT NULL DEFAULT 'free', -- 'free' | 'pro' | 'studio'
  status TEXT NOT NULL DEFAULT 'active',
  -- 'active' | 'past_due' | 'canceled' | 'trialing'
  stripe_customer_id TEXT UNIQUE,
  stripe_subscription_id TEXT UNIQUE,
  current_period_end TIMESTAMPTZ,
  cancel_at_period_end BOOLEAN NOT NULL DEFAULT FALSE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

ALTER TABLE subscriptions ENABLE ROW LEVEL SECURITY;
CREATE POLICY "subscriptions_own" ON subscriptions FOR SELECT USING (user_id = auth.uid());
CREATE POLICY "subscriptions_deny_write" ON subscriptions FOR ALL USING (false);
-- only service role (Rust webhook handler) writes subscriptions
```

---

### `notifications`

```sql
CREATE TABLE notifications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  type TEXT NOT NULL,
  title TEXT NOT NULL,
  body TEXT NOT NULL,
  entity_type TEXT, -- 'script' | 'comment' | 'review' | 'listen' | 'subscription' | etc.
  entity_id UUID,
  read_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_notifications_user ON notifications(user_id, created_at DESC);
CREATE INDEX idx_notifications_unread ON notifications(user_id, read_at) WHERE read_at IS NULL;

ALTER TABLE notifications ENABLE ROW LEVEL SECURITY;
CREATE POLICY "notifications_own" ON notifications FOR ALL USING (user_id = auth.uid());
```

---

### `notification_preferences`

```sql
CREATE TABLE notification_preferences (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  notification_type TEXT NOT NULL,
  in_app_enabled BOOLEAN NOT NULL DEFAULT TRUE,
  push_enabled BOOLEAN NOT NULL DEFAULT TRUE,
  UNIQUE(user_id, notification_type)
);

ALTER TABLE notification_preferences ENABLE ROW LEVEL SECURITY;
CREATE POLICY "notif_prefs_own" ON notification_preferences FOR ALL USING (user_id = auth.uid());
```

---

### `reports`

```sql
CREATE TABLE reports (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  reporter_id UUID NOT NULL REFERENCES profiles(id),
  entity_type TEXT NOT NULL, -- 'script' | 'profile' | 'review'
  entity_id UUID NOT NULL,
  reason TEXT NOT NULL,
  -- 'inappropriate_content' | 'spam' | 'copyright' | 'plagiarism'
  -- | 'harassment' | 'other'
  details TEXT,
  is_dmca BOOLEAN NOT NULL DEFAULT FALSE,
  status TEXT NOT NULL DEFAULT 'pending',
  -- 'pending' | 'actioned' | 'dismissed'
  priority TEXT NOT NULL DEFAULT 'normal', -- 'normal' | 'high'
  actioned_by UUID REFERENCES profiles(id),
  actioned_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE(reporter_id, entity_type, entity_id, reason)
);

CREATE INDEX idx_reports_status ON reports(status, priority, created_at DESC);

ALTER TABLE reports ENABLE ROW LEVEL SECURITY;
CREATE POLICY "reports_own_read" ON reports FOR SELECT USING (reporter_id = auth.uid());
CREATE POLICY "reports_insert" ON reports FOR INSERT WITH CHECK (reporter_id = auth.uid());
```

---

### `accessibility_preferences`

```sql
CREATE TABLE accessibility_preferences (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL UNIQUE REFERENCES profiles(id),
  high_contrast_mode BOOLEAN NOT NULL DEFAULT FALSE,
  reduced_motion BOOLEAN NOT NULL DEFAULT FALSE,
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

ALTER TABLE accessibility_preferences ENABLE ROW LEVEL SECURITY;
CREATE POLICY "accessibility_own" ON accessibility_preferences FOR ALL USING (user_id = auth.uid());
```

---

### `writers_room_sessions`

```sql
CREATE TABLE writers_room_sessions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  script_id UUID NOT NULL REFERENCES scripts(id),
  showrunner_id UUID NOT NULL REFERENCES profiles(id),
  script_locked BOOLEAN NOT NULL DEFAULT FALSE,
  started_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  ended_at TIMESTAMPTZ,
  muted_user_ids UUID[] NOT NULL DEFAULT '{}'
);

ALTER TABLE writers_room_sessions ENABLE ROW LEVEL SECURITY;
CREATE POLICY "wr_sessions_showrunner" ON writers_room_sessions FOR ALL
  USING (showrunner_id = auth.uid());
CREATE POLICY "wr_sessions_collaborator_read" ON writers_room_sessions FOR SELECT
  USING (EXISTS (SELECT 1 FROM collaborators WHERE script_id = writers_room_sessions.script_id AND user_id = auth.uid()));
```

---

### `beat_assignments`

```sql
CREATE TABLE beat_assignments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  script_id UUID NOT NULL REFERENCES scripts(id),
  beat_id UUID NOT NULL REFERENCES beat_board_cards(id),
  assigned_to UUID REFERENCES profiles(id),
  assigned_by UUID NOT NULL REFERENCES profiles(id),
  assigned_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE(beat_id)
);

ALTER TABLE beat_assignments ENABLE ROW LEVEL SECURITY;
CREATE POLICY "beat_assignments_showrunner" ON beat_assignments FOR ALL
  USING (assigned_by = auth.uid());
```

---

### `admin_audit_log`

```sql
CREATE TABLE admin_audit_log (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  admin_id UUID NOT NULL REFERENCES profiles(id),
  action TEXT NOT NULL,
  entity_type TEXT,
  entity_id UUID,
  reason TEXT,
  metadata JSONB,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_admin_audit_log_created ON admin_audit_log(created_at DESC);
CREATE INDEX idx_admin_audit_log_admin ON admin_audit_log(admin_id, created_at DESC);

ALTER TABLE admin_audit_log ENABLE ROW LEVEL SECURITY;
CREATE POLICY "deny_all_admin_audit_log" ON admin_audit_log FOR ALL USING (false);
-- only service role (Rust) writes/reads
```

---

### `personalised_feeds` (materialised cache)

```sql
CREATE TABLE personalised_feeds (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  publication_id UUID NOT NULL REFERENCES script_publications(id) ON DELETE CASCADE,
  score FLOAT NOT NULL,
  computed_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE(user_id, publication_id)
);

CREATE INDEX idx_personalised_feeds_user ON personalised_feeds(user_id, score DESC);

ALTER TABLE personalised_feeds ENABLE ROW LEVEL SECURITY;
CREATE POLICY "feeds_own" ON personalised_feeds FOR SELECT USING (user_id = auth.uid());
```

---

## DB Triggers

```sql
-- Auto-create profiles row on auth signup
CREATE OR REPLACE FUNCTION handle_new_user()
RETURNS TRIGGER LANGUAGE plpgsql SECURITY DEFINER AS $$
BEGIN
  INSERT INTO profiles (id, email)
  VALUES (NEW.id, NEW.email);
  INSERT INTO subscriptions (user_id) VALUES (NEW.id);
  RETURN NEW;
END;
$$;

CREATE TRIGGER on_auth_user_created
AFTER INSERT ON auth.users
FOR EACH ROW EXECUTE FUNCTION handle_new_user();

-- Update like_count on script_publications
CREATE OR REPLACE FUNCTION update_like_count()
RETURNS TRIGGER LANGUAGE plpgsql AS $$
BEGIN
  IF TG_OP = 'INSERT' THEN
    UPDATE script_publications SET like_count = like_count + 1 WHERE id = NEW.publication_id;
  ELSIF TG_OP = 'DELETE' THEN
    UPDATE script_publications SET like_count = GREATEST(0, like_count - 1) WHERE id = OLD.publication_id;
  END IF;
  RETURN NULL;
END;
$$;

CREATE TRIGGER trg_like_count
AFTER INSERT OR DELETE ON script_likes
FOR EACH ROW EXECUTE FUNCTION update_like_count();

-- Update follower/following counts
CREATE OR REPLACE FUNCTION update_follow_counts()
RETURNS TRIGGER LANGUAGE plpgsql AS $$
BEGIN
  IF TG_OP = 'INSERT' THEN
    UPDATE profiles SET follower_count = follower_count + 1 WHERE id = NEW.following_id;
    UPDATE profiles SET following_count = following_count + 1 WHERE id = NEW.follower_id;
  ELSIF TG_OP = 'DELETE' THEN
    UPDATE profiles SET follower_count = GREATEST(0, follower_count - 1) WHERE id = OLD.following_id;
    UPDATE profiles SET following_count = GREATEST(0, following_count - 1) WHERE id = OLD.follower_id;
  END IF;
  RETURN NULL;
END;
$$;

CREATE TRIGGER trg_follow_counts
AFTER INSERT OR DELETE ON user_follows
FOR EACH ROW EXECUTE FUNCTION update_follow_counts();
```

---

## Migration Order

Run migrations in this order to respect foreign key dependencies:

1. `profiles`
2. `subscriptions`
3. `scripts`
4. `blocks`
5. `collaborators`
6. `drafts`
7. `script_branches`
8. `comments`
9. `scene_intelligence`
10. `story_bible`
11. `agent_approvals`
12. `continuity_flags`
13. `writing_sessions`
14. `beat_board_cards`
15. `beat_assignments`
16. `shots`
17. `inline_tags`
18. `research_notes`
19. `formatting_preferences`
20. `user_follows`
21. `username_change_log`
22. `script_publications`
23. `script_likes`
24. `script_bookmarks`
25. `script_views`
26. `leaderboard_snapshots`
27. `listen_productions`
28. `listen_voice_assignments`
29. `tts_voice_cache`
30. `reviewer_profiles`
31. `payout_errors`
32. `reviews`
33. `producer_profiles`
34. `producer_saved_scripts`
35. `producer_contact_requests`
36. `notifications`
37. `notification_preferences`
38. `reports`
39. `accessibility_preferences`
40. `writers_room_sessions`
41. `admin_audit_log`
42. `personalised_feeds`
43. Triggers
