OUTPUT_BASENAME := ResumeSherzodKuchkarov2026

.PHONY: pdf clean

pdf:
	@for md in *.md; do \
		base="$${md%.md}"; \
		lang="$${base##*.}"; \
		if [ "$$lang" = "$$base" ]; then \
			lang=""; \
		fi; \
		mainfont="Helvetica Neue"; \
		defaults_opt=""; \
		if [ "$$lang" = "ja" ]; then \
			mainfont="Hiragino Sans W3"; \
			defaults_opt="--defaults=pandoc.ja.yaml"; \
		fi; \
		out="$(OUTPUT_BASENAME)$$lang.pdf"; \
		echo "Generating $$out from $$md"; \
		pandoc "$$md" \
			$$defaults_opt \
			--from=markdown+hard_line_breaks \
			--output="$$out" \
			--wrap=preserve \
			--pdf-engine=/Library/TeX/texbin/xelatex \
			-V geometry:margin=1in \
			-V mainfont="$$mainfont" \
			-V fontsize=10pt \
			-V linestretch=1.2 \
			-V linkcolor=blue \
			-V urlcolor=blue \
			-V documentclass=article \
			-V classoption=oneside \
			-V colorlinks=true; \
	done

clean:
	rm -f $(OUTPUT_BASENAME)*.pdf
