import subprocess

cmd = r"""cat files.txt | sed 's|gs://my-bucket/||' | parallel -j 8 '
  src="gs://my-bucket/{}";
  dest="./downloads/{}";
  mkdir -p $(dirname "$dest");
  gcloud storage cp "$src" "$dest"
'"""

subprocess.run(cmd, shell=True, check=True)
