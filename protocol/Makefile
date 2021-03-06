LATEXMK=latexmk --halt-on-error -bibtex -pdf
LATEX=pdflatex --halt-on-error

sprout.pdf: protocol.tex zcash.bib incremental_merkle.pdf key_components.pdf
	$(MAKE) pdf

sapling.pdf: protocol.tex zcash.bib incremental_merkle.pdf key_components_sapling.pdf
	$(MAKE) sapling

.PHONY: auxsprout
auxsprout:
	printf '\\renewcommand{\\docversion}{Version %s [\\SproutSpec]}' "$$(git describe --tags --abbrev=6)" |tee protocol.ver
	mkdir -p aux
	rm -f aux/sprout.* aux/protocol.*
	$(LATEXMK) -jobname=sprout -auxdir=aux -outdir=aux protocol

.PHONY: sprout
sprout:
	$(MAKE) auxsprout
	mv -f aux/sprout.pdf .

.PHONY: pvcsprout
pvcsprout:
	printf '\\renewcommand{\\docversion}{Version %s [\\SproutSpec]}' "$$(git describe --tags --abbrev=6)" |tee protocol.ver
	mkdir -p aux
	rm -f aux/sprout.* aux/protocol.*
	$(LATEXMK) -jobname=sprout -auxdir=aux -pvc protocol

.PHONY: auxsapling
auxsapling:
	printf '\\toggletrue{issapling}\n\\renewcommand{\\docversion}{Version %s [\\SaplingSpec]}' "$$(git describe --tags --abbrev=6)" |tee protocol.ver
	mkdir -p aux
	rm -f aux/sapling.*
	$(LATEXMK) -jobname=sapling -auxdir=aux -outdir=aux protocol

.PHONY: sapling
sapling:
	$(MAKE) auxsapling
	mv -f aux/sapling.pdf .
	cp -f sapling.pdf protocol.pdf

.PHONY: pvcsapling
pvcsapling:
	printf '\\toggletrue{issapling}\n\\renewcommand{\\docversion}{Version %s [\\SaplingSpec]}' "$$(git describe --tags --abbrev=6)" |tee protocol.ver
	mkdir -p aux
	rm -f aux/sapling.*
	$(LATEXMK) -jobname=sapling -auxdir=aux -pvc protocol

.PHONY: nolatexmk-sprout
nolatexmk-sprout:
	printf '\\renewcommand{\\docversion}{Version %s [\\SproutSpec]}' "$$(git describe --tags --abbrev=6)" |tee protocol.ver
	# If $(LATEX) fails, touch an input so that 'make' won't think it is up-to-date next time.
	rm -f sprout.aux sprout.bbl sprout.blg sprout.brf sprout.bcf
	$(LATEX) protocol.tex || { touch incremental_merkle.pdf; exit 1; }
	biber protocol
	$(LATEX) protocol.tex || { touch incremental_merkle.pdf; exit 1; }
	$(LATEX) protocol.tex || { touch incremental_merkle.pdf; exit 1; }
	$(LATEX) protocol.tex || { touch incremental_merkle.pdf; exit 1; }

.PHONY: nolatexmk-sapling
nolatexmk-sapling:
	printf '\\toggletrue{issapling}\n\\renewcommand{\\docversion}{Version %s [\\SaplingSpec]}' "$$(git describe --tags --abbrev=6)" |tee protocol.ver
	# If $(LATEX) fails, touch an input so that 'make' won't think it is up-to-date next time.
	rm -f sapling.aux sapling.bbl sapling.blg sapling.brf sapling.bcf
	$(LATEX) -jobname=sapling protocol.tex || { touch incremental_merkle.pdf; exit 1; }
	biber sapling
	$(LATEX) -jobname=sapling protocol.tex || { touch incremental_merkle.pdf; exit 1; }
	$(LATEX) -jobname=sapling protocol.tex || { touch incremental_merkle.pdf; exit 1; }
	$(LATEX) -jobname=sapling protocol.tex || { touch incremental_merkle.pdf; exit 1; }

.PHONY: html
html: sprout.pdf sapling.pdf
	# Can't use --split-pages 1 because XHR doesn't work by default on local files in Chrome.
	pdf2htmlEX --decompose-ligature 1 --font-size-multiplier 65 --fit-width 1000 --dest-dir html sprout.pdf
	pdf2htmlEX --decompose-ligature 1 --font-size-multiplier 65 --fit-width 1000 --dest-dir html sapling.pdf

.PHONY: clean
clean:
	rm -f aux/* html/* protocol.ver \
              sprout.dvi  sprout.pdf  sprout.bbl  sprout.blg  sprout.brf      sprout.toc \
              sprout.aux  sprout.out  sprout.log  sprout.bcf  sprout.run.xml  sprout.fls  sprout.fdb_latexmk \
              sapling.dvi sapling.pdf sapling.bbl sapling.blg sapling.brf     sapling.toc \
              sapling.aux sapling.out sapling.log sapling.bcf sapling.run.xml sapling.fls sapling.fdb_latexmk

optimizer-installed.flag:
	# Nail down git commits to make backdooring somewhat harder.
	git clone https://github.com/pts/sam2p.git
	cd sam2p && git reset --hard a2d7819107324faf7b0904fc7074f7dd4a0e16c7 && $(MAKE)
	git clone https://github.com/pts/tif22pnm.git
	cd tif22pnm && git reset --hard 22217c1a3ea355a899e9c7c79903488ca13d1dfe && $(MAKE)
	git clone https://github.com/pts/pdfsizeopt.git
	cd pdfsizeopt && git reset --hard 47a03403d70f6975888cee966858bebc51b76463
	touch optimizer-installed.flag

.PHONY: clean-optimizer
clean-optimizer:
	rm -rf sam2p tif22pnm pdfsizeopt optimizer-installed.flag

.PHONY: optsprout
optsprout: optimizer-installed.flag
	$(MAKE) auxsprout
	PATH="${PATH}:$(CURDIR)/sam2p:$(CURDIR)/tif22pnm" pdfsizeopt/pdfsizeopt --v=40 --use-image-optimizer=sam2p \
              --tmp-dir=aux aux/sprout.pdf sprout.pdf

.PHONY: optsapling
optsapling: optimizer-installed.flag
	$(MAKE) auxsapling
	PATH="${PATH}:$(CURDIR)/sam2p:$(CURDIR)/tif22pnm" pdfsizeopt/pdfsizeopt --v=40 --use-image-optimizer=sam2p \
              --tmp-dir=aux aux/sapling.pdf sapling.pdf
	cp -f sapling.pdf protocol.pdf

.PHONY: optimized
optimized: optsprout optsapling
