all:doc
.PHONY:doc

doc:report.tex
	xelatex report.tex
	bibtex report.aux
	xelatex report.tex
	xelatex report.tex
	xelatex report.tex
clean:
	rm -rf *.aux
	rm -rf *.bbl *.blg *.log *.out *.toc *.nav *.snm *.loc *.soc
distclean:clean
	rm -f *.pdf
