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