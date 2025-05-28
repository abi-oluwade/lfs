#!/bin/bash

OUTPUT="tools.md"

echo "# List of Executable Tools in \$PATH with Versions" > "$OUTPUT"
echo "" >> "$OUTPUT"
echo "| Tool | Version |" >> "$OUTPUT"
echo "|------|---------|" >> "$OUTPUT"

# Save PATH dirs to a variable and loop
IFS=':'
for dir in $PATH; do
  if [[ -d "$dir" ]]; then
    find "$dir" -maxdepth 1 -executable -type f 2>/dev/null
  fi
done | sort -u | while read -r fullpath; do
  tool=$(basename "$fullpath")
  # Try getting version info
  version=$("$tool" --version 2>/dev/null | head -n 1)
  if [[ -z "$version" ]]; then
    version=$("$tool" -v 2>/dev/null | head -n 1)
  fi
  [[ -z "$version" ]] && version="(unknown)"
  # Escape pipes for Markdown tables
  version="${version//|/\\|}"
  echo "| \`$tool\` | $version |" >> "$OUTPUT"
done