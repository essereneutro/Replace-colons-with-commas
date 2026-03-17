# Replace colons with commas

A batch renaming script.

On macOS, folder names that appear with a slash (`/`) may actually contain a colon (`:`). When transferring those folders to a Windows computer, this can cause problems because Windows does not allow colons in file or folder names. This script renames those folders and its subfolders by replacing colons (`:`) with commas (`,`).

## Instructions

Open Terminal, navigate to the target folder using `cd folder_name` (or drag and drop the folder into Terminal, if supported), press Enter, then paste and run the following code:

```bash
find . -depth -type d ! -name "._*" | while IFS= read -r dir; do
    [ "$dir" = "." ] && continue

    base="${dir##*/}"
    parent="${dir%/*}"

    nb="${base// :/,}"
    nb="${nb//:/,}"
    [ "$base" = "$nb" ] && continue

    final="${parent}/${nb}"

    i=0
    while :; do
        if [ $i -eq 0 ]; then
            tmp="${final}_TMP_$$"
        else
            tmp="${final}_TMP_$$.$i"
        fi
        [ -e "$tmp" ] || break
        i=$((i+1))
    done

    if ! mv -- "$dir" "$tmp"; then
        printf "[PHASE 1 ERROR] ORIG: \"%s\" -> TMP: \"%s\" (target: \"%s\")\n" \
            "$dir" "$tmp" "$final" | tee -a rename_errors.log
        continue
    fi

    if [ -e "$final" ]; then
        printf "[SKIP] Target already exists: \"%s\" -> \"%s\"\n" \
            "$dir" "$final" | tee -a rename_errors.log
        mv -- "$tmp" "$dir"
        continue
    fi

    if ! mv -- "$tmp" "$final"; then
        printf "[PHASE 2 ERROR] TMP: \"%s\" -> TARGET: \"%s\" (original: \"%s\")\n" \
            "$tmp" "$final" "$dir" | tee -a rename_errors.log
        mv -- "$tmp" "$dir"
    fi
done
