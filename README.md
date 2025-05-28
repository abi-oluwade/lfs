# lfs

#!/bin/bash

# Output file
OUTPUT="packages.md"

# Header
echo "# List of Available Ubuntu Packages" > "$OUTPUT"
echo "" >> "$OUTPUT"
echo "**Generated on $(date)**" >> "$OUTPUT"
echo "" >> "$OUTPUT"

# Markdown table header
echo "| Package Name | Version | Description |" >> "$OUTPUT"
echo "|--------------|---------|-------------|" >> "$OUTPUT"

# Extract data and format as table rows
apt-cache dumpavail | awk '
  /^Package:/ { pkg=$2 }
  /^Version:/ { ver=$2 }
  /^Description:/ {
    desc=$0
    sub(/^Description: /, "", desc)
    # Escape pipe characters in description
    gsub(/\|/, "\\|", desc)
    printf("| %s | %s | %s |\n", pkg, ver, desc)
  }
' >> "$OUTPUT"

#!/bin/bash

OUTPUT="available-tools.md"

# Write header
{
  echo "# List of Available Executables in \$PATH"
  echo ""
  echo "**Generated on $(date)**"
  echo ""
  echo "| Tool | Version |"
  echo "|------|---------|"
} > "$OUTPUT"

# Temporary file to store names
TMPFILE=$(mktemp)

# Collect all unique executables in PATH
IFS=":"
for dir in $PATH; do
  [[ -d "$dir" ]] && find "$dir" -maxdepth 1 -type f -executable 2>/dev/null
done | awk -F/ '{print $NF}' | sort -u > "$TMPFILE"

# Try to get version info
while read -r tool; do
  cmd_path=$(command -v "$tool" 2>/dev/null)
  if [[ -x "$cmd_path" ]]; then
    # Try common version flags
    version=$(
      "$tool" --version 2>/dev/null | head -n 1 ||
      "$tool" -v 2>/dev/null | head -n 1
    )
    # If no version output, mark as unknown
    [[ -z "$version" ]] && version="(version unknown)"
    # Escape pipe for Markdown
    version="${version//|/\\|}"
    echo "| \`$tool\` | $version |" >> "$OUTPUT"
  fi
done < "$TMPFILE"

# Cleanup
rm "$TMPFILE"