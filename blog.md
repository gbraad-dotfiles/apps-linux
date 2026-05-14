# Blog Management

Blog post management tool with ideas, drafts, and publishing workflow.

### config
```ini
[blog]
  dir=${HOME}/Projects/gbraad-pages/blog-content
  port=2080
```

### checkout
```sh
if [ -d "$BLOG_DIR" ]; then
    echo "Blog directory already exists: $BLOG_DIR"
    exit 1
fi

echo "Setting up blog directory structure..."
echo ""

# Create main directory
mkdir -p "$BLOG_DIR"

# Create subdirectories
mkdir -p "$BLOG_DIR/todo"
mkdir -p "$BLOG_DIR/draft"
mkdir -p "$BLOG_DIR/done"

# Create README
cat > "$BLOG_DIR/README.md" <<'EOF'
# Blog Content

Blog post management with ideas, drafts, and published posts.

## Structure

- `todo/` - Blog post ideas and outlines
- `draft/` - Work in progress posts
- `done/` - Published posts tracking
- `*.md` - Published posts

## Usage

Use the `blog` actionfile:

```bash
app blog stats      # Show statistics
app blog ideas      # Browse ideas
app blog drafts     # Edit drafts
app blog new        # Create new post
app blog publish    # Publish a draft
```
EOF

# Create template
cat > "$BLOG_DIR/todo/template.md" <<'EOF'
# Blog Post Template

## Post Title Here

**Why**: Why this post matters

**Audience**: Who should read this

**Length**: Estimated word count

**Category**: Technical / Leadership / Career / Tutorial

**Tags**: tag1, tag2, tag3

**Key points**:
- Point 1
- Point 2
- Point 3
- Conclusion
EOF

echo "- Created directory: $BLOG_DIR"
echo "- Created subdirectories: todo/, draft/, done/"
echo "- Created README.md"
echo "- Created template: todo/template.md"
echo ""
echo "Blog directory is ready!"
echo ""
echo "Next steps:"
echo "  1. Add your blog post ideas to todo/"
echo "  2. Use 'app blog new' to create posts"
echo "  3. Use 'app blog stats' to see progress"
```

### info
```sh
echo "Blog Configuration"
echo "=================="
echo ""
echo "Blog Directory: $BLOG_DIR"

if [ -d "$BLOG_DIR" ]; then
    echo "Directory exists"
    echo ""
    echo "Structure:"
    [ -d "$BLOG_DIR/todo" ] && echo "todo/" || echo "no todo/"
    [ -d "$BLOG_DIR/draft" ] && echo "draft/" || echo "no draft/"
    [ -d "$BLOG_DIR/done" ] && echo "done/" || echo "no done/"
    
    echo ""
    echo "Contents:"
    echo "  Ideas:     $(find "$BLOG_DIR/todo" -name "*.md" -type f 2>/dev/null | grep -v -E "(template|README)" | wc -l) files"
    echo "  Drafts:    $(find "$BLOG_DIR/draft" -name "*.md" -type f 2>/dev/null | wc -l) posts"
    echo "  Published: $(find "$BLOG_DIR" -maxdepth 1 -name "*.md" -type f 2>/dev/null | grep -v -E "(README|SUMMARY|template)" | wc -l) posts"
else
    echo "Directory not found"
    echo ""
    echo "Run 'app blog checkout' to set up the directory structure"
fi
```

### stats
```sh
echo "Blog Statistics"
echo "==============="
echo ""

# Count ideas (h2 headings in todo files)
ideas=0
if [ -d "$BLOG_DIR/todo" ]; then
    for file in "$BLOG_DIR/todo"/*.md; do
        [ -f "$file" ] || continue
        [[ "$(basename "$file")" =~ ^(template|README) ]] && continue
        count=$(grep "^## " "$file" 2>/dev/null | wc -l)
        notes=$(grep "^## Notes" "$file" 2>/dev/null | wc -l)
        file_ideas=$((count - notes))
        ideas=$((ideas + file_ideas))
    done
fi

# Count drafts
drafts=0
if [ -d "$BLOG_DIR/draft" ]; then
    drafts=$(find "$BLOG_DIR/draft" -name "*.md" -type f 2>/dev/null | wc -l)
fi

# Count published
published=0
for file in "$BLOG_DIR"/*.md; do
    [ -f "$file" ] || continue
    [[ "$(basename "$file")" =~ ^(README|SUMMARY|template) ]] && continue
    published=$((published + 1))
done

echo "Ideas:     $ideas"
echo "Drafts:    $drafts"
echo "Published: $published"
echo ""

if [ $ideas -gt 0 ]; then
    echo "You have $ideas blog post ideas to work on"
fi

if [ $drafts -gt 0 ]; then
    echo "You have $drafts drafts in progress"
fi

if [ $published -gt 0 ]; then
    echo "You have published $published posts"
fi
```

### ideas
```sh
if [ ! -d "$BLOG_DIR/todo" ]; then
    echo "Error: todo directory not found"
    exit 1
fi

# Build list of idea files
files=()
for file in "$BLOG_DIR/todo"/*.md; do
    [ -f "$file" ] || continue
    [[ "$(basename "$file")" =~ ^(template|README) ]] && continue
    
    filename=$(basename "$file" .md)
    title=$(echo "$filename" | tr '-' ' ' | sed 's/\b\(.\)/\u\1/g')
    count=$(grep "^## " "$file" 2>/dev/null | wc -l)
    notes=$(grep "^## Notes" "$file" 2>/dev/null | wc -l)
    idea_count=$((count - notes))
    
    files+=("$title ($idea_count ideas)|$file")
done

# Use fzf to select and preview
selected=$(printf '%s\n' "${files[@]}" | fzf \
    --delimiter='|' \
    --with-nth=1 \
    --preview='cat {2}' \
    --preview-window=right:60%:wrap \
    --header='Select blog idea file to view (ESC to cancel)')

if [ -n "$selected" ]; then
    filepath=$(echo "$selected" | cut -d'|' -f2)
    cat "$filepath"
fi
```

### priority
```sh
if [ -f "$BLOG_DIR/todo/priority-high.md" ]; then
    cat "$BLOG_DIR/todo/priority-high.md"
else
    echo "No priority-high.md file found"
fi
```

### todos
```sh
output_file="$BLOG_DIR/todo/topics.md"
temp_file="/tmp/blog-topics-$$.md"

echo "Generating blog topics index..."
echo ""

# Start building the index
cat > "$temp_file" <<HEADER
# Blog Topics Index

Comprehensive index of all blog post ideas, organized by priority and category.

**Last Updated**: $(date +%Y-%m-%d)

---

HEADER

# Add high priority section
if [ -f "$BLOG_DIR/todo/priority-high.md" ]; then
    cat >> "$temp_file" <<'PRIORITY_HEADER'
## High Priority Posts

These posts should be written FIRST (before July 2026 job search).

PRIORITY_HEADER

    # Extract high priority posts
    grep "^## [0-9]" "$BLOG_DIR/todo/priority-high.md" | while IFS= read -r line; do
        # Extract title (remove number prefix and stars)
        title=$(echo "$line" | sed 's/^## [0-9]*\. *//; s/ ⭐.*$//')
        # Extract stars for priority
        stars=$(echo "$line" | grep -o '⭐*' | tail -1)
        
        # Create anchor link (lowercase, replace spaces with hyphens)
        anchor=$(echo "$title" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/--*/-/g' | sed 's/^-//; s/-$//')
        
        echo "- [$title](priority-high.md#$anchor) $stars" >> "$temp_file"
    done
    
    echo "" >> "$temp_file"
fi

# Add table of contents header
cat >> "$temp_file" <<'TOC'
## Categories

TOC

# Build categories section
for file in "$BLOG_DIR/todo"/*.md; do
    [ -f "$file" ] || continue
    filename=$(basename "$file")
    
    # Skip special files
    [[ "$filename" =~ ^(template|README|topics|priority-high) ]] && continue
    
    # Get file title (from first # heading or convert filename)
    file_title=$(grep -m 1 "^# " "$file" 2>/dev/null | sed 's/^# //')
    if [ -z "$file_title" ]; then
        file_title=$(echo "${filename%.md}" | tr '-' ' ' | sed 's/\b\(.\)/\u\1/g')
    fi
    
    # Count topics in file
    count=$(grep "^## " "$file" 2>/dev/null | grep -v "^## Notes" | wc -l)
    
    if [ "$count" -gt 0 ]; then
        # Add to TOC
        echo "- [$file_title](./$filename) ($count topics)" >> "$temp_file"
    fi
done

echo "" >> "$temp_file"
echo "---" >> "$temp_file"
echo "" >> "$temp_file"

# Add detailed category sections
for file in "$BLOG_DIR/todo"/*.md; do
    [ -f "$file" ] || continue
    filename=$(basename "$file")
    
    # Skip special files
    [[ "$filename" =~ ^(template|README|topics|priority-high) ]] && continue
    
    # Get file title
    file_title=$(grep -m 1 "^# " "$file" 2>/dev/null | sed 's/^# //')
    if [ -z "$file_title" ]; then
        file_title=$(echo "${filename%.md}" | tr '-' ' ' | sed 's/\b\(.\)/\u\1/g')
    fi
    
    # Count topics in file
    count=$(grep "^## " "$file" 2>/dev/null | grep -v "^## Notes" | wc -l)
    
    if [ "$count" -gt 0 ]; then
        # Add category heading
        echo "### [$file_title](./$filename)" >> "$temp_file"
        echo "" >> "$temp_file"
        
        # Extract topics from this file
        grep "^## " "$file" 2>/dev/null | while IFS= read -r topic_line; do
            # Skip Notes section
            [[ "$topic_line" =~ ^##\ Notes ]] && continue
            
            # Extract topic title
            topic_title=$(echo "$topic_line" | sed 's/^## *//')
            
            # Create anchor for this topic
            topic_anchor=$(echo "$topic_title" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/--*/-/g' | sed 's/^-//; s/-$//')
            
            # Add to category section
            echo "- [$topic_title](./$filename#$topic_anchor)" >> "$temp_file"
        done
        
        echo "" >> "$temp_file"
    fi
done

# Add statistics footer
cat >> "$temp_file" <<FOOTER

---

## Statistics

FOOTER

# Count total ideas
total_ideas=0
total_files=0
for file in "$BLOG_DIR/todo"/*.md; do
    [ -f "$file" ] || continue
    filename=$(basename "$file")
    [[ "$filename" =~ ^(template|README|topics) ]] && continue
    
    count=$(grep "^## " "$file" 2>/dev/null | wc -l)
    notes=$(grep "^## Notes" "$file" 2>/dev/null | wc -l)
    file_ideas=$((count - notes))
    
    if [ "$file_ideas" -gt 0 ]; then
        total_ideas=$((total_ideas + file_ideas))
        total_files=$((total_files + 1))
    fi
done

high_priority=$(grep -c "^## [0-9]" "$BLOG_DIR/todo/priority-high.md" 2>/dev/null || echo 0)

cat >> "$temp_file" <<STATS
- Total Topics: $total_ideas
- Total Files: $total_files
- High Priority: $high_priority

---

**Generated by**: \`app blog todos\`
STATS

# Move temp file to final location
mv "$temp_file" "$output_file"

echo "- Created: $output_file"
echo ""
echo "Topics indexed:"
echo "  Total topics: $total_ideas"
echo "  Files processed: $total_files"
echo "  High priority: $high_priority"
echo ""
echo "View: cat $output_file"
echo "Or: app blog ideas (then select topics.md)"
```

### drafts
```sh
if [ ! -d "$BLOG_DIR/draft" ]; then
    echo "No draft directory found"
    exit 1
fi

# Find all drafts
drafts=()
for file in "$BLOG_DIR/draft"/*.md; do
    [ -f "$file" ] || continue
    
    filename=$(basename "$file")
    title=$(grep -m 1 "^title:" "$file" 2>/dev/null | sed 's/^title: *//; s/"//g' || echo "No title")
    date=$(grep -m 1 "^date:" "$file" 2>/dev/null | sed 's/^date: *//' || echo "")
    
    if [ -n "$date" ]; then
        drafts+=("$filename - $title [$date]|$file")
    else
        drafts+=("$filename - $title|$file")
    fi
done

if [ ${#drafts[@]} -eq 0 ]; then
    echo "No drafts found"
    exit 0
fi

# Use fzf to select and preview
selected=$(printf '%s\n' "${drafts[@]}" | fzf \
    --delimiter='|' \
    --with-nth=1 \
    --preview='cat {2}' \
    --preview-window=right:60%:wrap \
    --header='Select draft to edit (ESC to cancel)')

if [ -n "$selected" ]; then
    filepath=$(echo "$selected" | cut -d'|' -f2)
    ${EDITOR:-vim} "$filepath"
fi
```

### published
```sh
# Find all published posts
posts=()
for file in "$BLOG_DIR"/*.md; do
    [ -f "$file" ] || continue
    [[ "$(basename "$file")" =~ ^(README|SUMMARY|template) ]] && continue
    
    filename=$(basename "$file")
    title=$(grep -m 1 "^title:" "$file" 2>/dev/null | sed 's/^title: *//; s/"//g' || echo "No title")
    date=$(grep -m 1 "^date:" "$file" 2>/dev/null | sed 's/^date: *//' || echo "")
    
    if [ -n "$date" ]; then
        posts+=("$filename - $title [$date]|$file")
    else
        posts+=("$filename - $title|$file")
    fi
done

if [ ${#posts[@]} -eq 0 ]; then
    echo "No published posts found"
    exit 0
fi

# Use fzf to select and preview
selected=$(printf '%s\n' "${posts[@]}" | fzf \
    --delimiter='|' \
    --with-nth=1 \
    --preview='cat {2}' \
    --preview-window=right:60%:wrap \
    --header='Select post to view/edit (ESC to cancel)')

if [ -n "$selected" ]; then
    filepath=$(echo "$selected" | cut -d'|' -f2)
    
    # Ask what to do
    action=$(echo -e "View\nEdit" | fzf --header="What do you want to do?")
    
    if [ "$action" = "View" ]; then
        cat "$filepath"
    elif [ "$action" = "Edit" ]; then
        ${EDITOR:-vim} "$filepath"
    fi
fi
```

### new
```sh
# Get title
echo "Post title:"
read -r post_title

if [ -z "$post_title" ]; then
    echo "Error: Title is required"
    exit 1
fi

# Generate slug
slug=$(echo "$post_title" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/--*/-/g' | sed 's/^-//; s/-$//')

# Find next number
next_num=0
for file in "$BLOG_DIR"/*.md "$BLOG_DIR/draft"/*.md; do
    [ -f "$file" ] || continue
    num=$(basename "$file" | grep -o '^[0-9]*' || echo 0)
    if [ "$num" -gt "$next_num" ]; then
        next_num=$num
    fi
done
next_num=$((next_num + 1))
next_num=$(printf "%04d" $next_num)

# Select category with fzf
category=$(echo -e "Technical\nLeadership\nCareer\nAbout me\nTutorial\nOpinion" | fzf --header="Select category")

if [ -z "$category" ]; then
    echo "Error: Category is required"
    exit 1
fi

# Get tags
echo "Tags (comma-separated):"
read -r tags

# Select destination with fzf
destination=$(echo -e "draft\npublished" | fzf --header="Save to draft or publish directly?")

# Create file path
filename="${next_num}-${slug}.md"
if [ "$destination" = "draft" ]; then
    filepath="$BLOG_DIR/draft/$filename"
else
    filepath="$BLOG_DIR/$filename"
fi

# Get current date
today=$(date +%Y-%m-%d)

# Create post from template
cat > "$filepath" <<EOF
---
title: "$post_title"
date: $today
category: $category
tags: [$tags]
---

# $post_title

## Introduction

[Write your introduction here. Why does this topic matter? What will readers learn?]

## Background

[Provide context and background information]

## Main Content

[Your main points and detailed content]

### Subsection 1

[Content here]

### Subsection 2

[Content here]

## Conclusion

[Summarize key points and takeaways]

## Call to Action

[Optional: What should readers do next? Leave a comment? Try something? Share?]

---

**About this post**: [Optional meta-information about writing process, updates, etc.]
EOF

echo ""
echo "- Created: $filepath"
echo ""

# Ask if user wants to edit now
edit_now=$(echo -e "Yes\nNo" | fzf --header="Edit now?")

if [ "$edit_now" = "Yes" ]; then
    ${EDITOR:-vim} "$filepath"
fi
```

### edit
```sh
# Find all posts (drafts and published)
posts=()

# Add drafts
if [ -d "$BLOG_DIR/draft" ]; then
    for file in "$BLOG_DIR/draft"/*.md; do
        [ -f "$file" ] || continue
        filename=$(basename "$file")
        title=$(grep -m 1 "^title:" "$file" 2>/dev/null | sed 's/^title: *//; s/"//g' || echo "No title")
        posts+=("[DRAFT] $filename - $title|$file")
    done
fi

# Add published
for file in "$BLOG_DIR"/*.md; do
    [ -f "$file" ] || continue
    [[ "$(basename "$file")" =~ ^(README|SUMMARY|template) ]] && continue
    filename=$(basename "$file")
    title=$(grep -m 1 "^title:" "$file" 2>/dev/null | sed 's/^title: *//; s/"//g' || echo "No title")
    posts+=("[PUB] $filename - $title|$file")
done

if [ ${#posts[@]} -eq 0 ]; then
    echo "No posts found"
    exit 0
fi

# Use fzf to select
selected=$(printf '%s\n' "${posts[@]}" | fzf \
    --delimiter='|' \
    --with-nth=1 \
    --preview='cat {2}' \
    --preview-window=right:60%:wrap \
    --header='Select post to edit (ESC to cancel)')

if [ -n "$selected" ]; then
    filepath=$(echo "$selected" | cut -d'|' -f2)
    ${EDITOR:-vim} "$filepath"
fi
```

### publish
```sh
if [ ! -d "$BLOG_DIR/draft" ]; then
    echo "No draft directory found"
    exit 1
fi

# Find all drafts
drafts=()
for file in "$BLOG_DIR/draft"/*.md; do
    [ -f "$file" ] || continue
    filename=$(basename "$file")
    title=$(grep -m 1 "^title:" "$file" 2>/dev/null | sed 's/^title: *//; s/"//g' || echo "No title")
    drafts+=("$filename - $title|$file")
done

if [ ${#drafts[@]} -eq 0 ]; then
    echo "No drafts to publish"
    exit 0
fi

# Use fzf to select
selected=$(printf '%s\n' "${drafts[@]}" | fzf \
    --delimiter='|' \
    --with-nth=1 \
    --preview='cat {2}' \
    --preview-window=right:60%:wrap \
    --header='Select draft to publish (ESC to cancel)')

if [ -n "$selected" ]; then
    filepath=$(echo "$selected" | cut -d'|' -f2)
    filename=$(basename "$filepath")
    dest="$BLOG_DIR/$filename"
    
    # Confirm
    confirm=$(echo -e "Yes\nNo" | fzf --header="Publish $filename?")
    
    if [ "$confirm" = "Yes" ]; then
        mv "$filepath" "$dest"
        echo ""
        echo "Published: $filename"
        echo "Location: $dest"
        echo ""
        echo "Next steps:"
        echo "  1. Update SUMMARY.md if needed"
        echo "  2. Git commit: git add $filename SUMMARY.md"
        echo "  3. Git commit: git commit -m 'Add blog post: $filename'"
        echo "  4. Push: git push"
    fi
fi
```

### search
```sh
# Search all markdown files
if command -v rg &> /dev/null; then
    # Use ripgrep if available
    cd "$BLOG_DIR" && rg --line-number --color=always --smart-case "" | fzf \
        --ansi \
        --delimiter=':' \
        --preview='bat --color=always --highlight-line={2} {1} 2>/dev/null || cat {1}' \
        --preview-window=right:60%:wrap:+{2}/2 \
        --header='Search blog content (type to filter)' \
        --bind='enter:execute(${EDITOR:-vim} {1} +{2})'
else
    # Fallback to grep
    grep -r -n -H --color=always "" "$BLOG_DIR"/*.md "$BLOG_DIR"/draft/*.md "$BLOG_DIR"/todo/*.md 2>/dev/null | fzf \
        --ansi \
        --delimiter=':' \
        --preview='cat {1}' \
        --preview-window=right:60%:wrap \
        --header='Search blog content (type to filter)' \
        --bind='enter:execute(${EDITOR:-vim} {1})'
fi
```

### browse
```sh
while true; do
    action=$(echo -e "Ideas\nDrafts\nPublished\nStats\nBuild\nServe\nBuild & Serve\nExit" | fzf --header="Browse Blog Content")
    
    case "$action" in
        "Ideas")
            app blog ideas
            ;;
        "Drafts")
            app blog drafts
            ;;
        "Published")
            app blog published
            ;;
        "Stats")
            app blog stats
            echo ""
            read -p "Press Enter to continue..."
            ;;
        "Build")
            app blog build
            echo ""
            read -p "Press Enter to continue..."
            ;;
        "Serve")
            app blog serve
            ;;
        "Build & Serve")
            app blog build-serve
            ;;
        "Exit"|"")
            break
            ;;
    esac
done
```

### quick
```sh
echo "=== Blog Quick Start ==="
echo ""

# Show stats
app blog stats

echo ""
echo "=== High Priority Posts ==="
echo ""

# Show priority items
if [ -f "$BLOG_DIR/todo/priority-high.md" ]; then
    grep "^## " "$BLOG_DIR/todo/priority-high.md" | grep -v "^## Notes" | head -5
fi

echo ""
echo ""

# Ask what to do
action=$(echo -e "Create new post\nView all ideas\nEdit draft\nBuild & serve\nNothing, just checking" | fzf --header="What would you like to do?")

case "$action" in
    "Create new post")
        app blog new
        ;;
    "View all ideas")
        app blog ideas
        ;;
    "Edit draft")
        app blog drafts
        ;;
    "Build & serve")
        app blog build-serve
        ;;
esac
```

### build
```sh
if [ ! -d "$BLOG_DIR" ]; then
    echo "Error: Blog directory not found: $BLOG_DIR"
    exit 1
fi

cd "$BLOG_DIR"

if [ -x "./build.sh" ]; then
    echo "Building blog..."
    ./build.sh
elif [ -f "Makefile" ]; then
    echo "Building blog..."
    make
else
    echo "No build script found (looking for build.sh or Makefile)"
    echo "Directory: $BLOG_DIR"
    exit 1
fi
```

### serve
```sh
if [ ! -d "$BLOG_DIR" ]; then
    echo "Error: Blog directory not found: $BLOG_DIR"
    exit 1
fi

cd "$BLOG_DIR"

echo "Starting HTTP server in $BLOG_DIR"
echo "Server running at: http://localhost:$BLOG_PORT"
echo ""
echo "Press Ctrl+C to stop"
echo ""

python3 -m http.server "$BLOG_PORT"
```

### build-serve
```sh
echo "Building blog..."
app blog build

if [ $? -eq 0 ]; then
    echo ""
    echo "Build successful! Starting server..."
    echo ""
    app blog serve
else
    echo "Build failed"
    exit 1
fi
```

### default alias run
```sh
app blog quick
```
