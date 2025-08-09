# Aaron Aycock's Blog

Personal blog about Wi-Fi, IoT, and technology. Built with Jekyll and hosted on GitHub Pages.

**Live site:** [https://blog.aaronaycock.com](https://blog.aaronaycock.com)

## Creating a New Blog Post

### 1. Create the Post File

Create a new markdown file in the `_posts/` directory with the naming format:
```
YYYY-MM-DD-post-title.markdown
```

Example: `2025-01-09-my-new-post.markdown`

### 2. Add Front Matter

Start your post with the required front matter:

```yaml
---
title:  "Your Post Title Here"
date:   2025-01-09 10:00:00
categories: [category1, category2]
tags: [tag1, tag2, tag3]
---
```

### 3. Write Your Content

Add your post content below the front matter using Markdown formatting:

```markdown
## Introduction
Your introduction text here...

## Main Content
Your main content here...

### Code Examples
```python
# Your code here
print("Hello, World!")
```

### Images
![Alt text](/images/your-image.png)
```

### 4. Test Locally

Before pushing, test your post locally:

```bash
# Install dependencies (first time only)
bundle install

# Run the Jekyll server
bundle exec jekyll serve --watch

# View at http://localhost:4000
```

### 5. Add Images (if needed)

Place any images in the `/images/` directory and reference them in your post:
```markdown
![Description](/images/your-image-name.png)
```

### 6. Commit and Push

```bash
# Add your new post
git add _posts/YYYY-MM-DD-your-post-title.markdown

# Add any images
git add images/your-image.png

# Commit with a descriptive message
git commit -m "Add blog post: Your Post Title"

# Push to GitHub
git push origin main
```

The site will automatically rebuild and your post will be live in a few minutes!

## Post Categories & Tags

Common categories used:
- `wireless`
- `raspberry-pi`
- `security`
- `networking`
- `iot`
- `homelab`

Popular tags:
- `wifi`
- `hostapd`
- `monitoring`
- `tutorial`
- `homeassistant`
- `linux`

## Tips

- Keep post titles concise and descriptive
- Use categories sparingly (1-2 per post)
- Use tags more liberally (3-5 per post)
- Include code examples when relevant
- Optimize images before uploading (use PNG for screenshots, JPEG for photos)
- Preview locally before pushing to catch any formatting issues

## Site Configuration

Main configuration is in `_config.yml`. Key settings:
- Site title and description
- Author information
- Google Analytics tracking
- Pagination settings

## Theme

This blog uses the Jekyll-Uno theme, a minimal responsive theme based on the Uno theme for Ghost.

## License

Content is under [MIT License](/LICENSE).