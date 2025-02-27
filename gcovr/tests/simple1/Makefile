
BASE_OS:=$(shell uname | cut -d'-' -f1)
ifneq (,$(findstring MINGW,$(BASE_OS)))
  BASE_OS:=MSYS_NT
endif

ifeq ($(BASE_OS),MSYS_NT)
  FAIL_UNDER_LINE := 76.9
else
  FAIL_UNDER_LINE := $(shell cat reference/$(CC)/fail_under)
endif

all:
	$(CXX) -fprofile-arcs -ftest-coverage -fPIC main.cpp -o testcase

run: txt xml html sonarqube json json_summary coveralls

txt:
	./testcase
	# a couple of tests about failure thresholds
	$(GCOVR) --fail-under-line 80.1 2>fail_under.stderr; test $$? -eq 2 || ( cat fail_under.stderr & exit 1 )
	grep -F "failed minimum line coverage" fail_under.stderr
	$(GCOVR) --fail-under-branch 50.1 2>fail_under.stderr; test $$? -eq 4 || ( cat fail_under.stderr & exit 1 )
	grep -F "failed minimum branch coverage" fail_under.stderr
	$(GCOVR) --fail-under-line 80.1 --fail-under-branch 50.1 2>fail_under.stderr; test $$? -eq 6 || ( cat fail_under.stderr & exit 1 )
	grep -F "failed minimum line coverage" fail_under.stderr
	grep -F "failed minimum branch coverage" fail_under.stderr
	$(GCOVR) --fail-under-line $(FAIL_UNDER_LINE) --fail-under-branch 49.9 2>fail_under.stderr; test $$? -eq 0 || ( cat fail_under.stderr & exit 1 )
	grep -F "failed minimum line coverage" fail_under.stderr; test $$? -eq 1
	grep -F "failed minimum branch coverage" fail_under.stderr; test $$? -eq 1
	# generate actual output
	$(GCOVR) -d --txt -o coverage.txt

xml:
	./testcase
	$(GCOVR) -d -x -o coverage.xml

html:
	./testcase
	# these test cases also cover some CSS-linking permutations
	$(GCOVR) --html coverage-summary-includecss.html
	$(GCOVR) --html coverage-summary-linkcss.html --no-html-self-contained
	$(GCOVR) --html-details coverage-details-includecss.html --html-self-contained
	$(GCOVR) --html-details coverage-details-linkcss.html

sonarqube:
	./testcase
	$(GCOVR) -d --sonarqube sonarqube.xml


json_summary:
	./testcase
	$(GCOVR) -d --json-summary-pretty -o summary_coverage.json

json:
	./testcase
	$(GCOVR) --json coverage.json
	
coveralls:
	./testcase
	$(GCOVR) -d --coveralls coveralls.json

clean:
	rm -f testcase
	rm -f fail_under.stderr
	rm -f *.gc*
	rm -f coverage.txt coverage.xml coverage*.html coverage*.css sonarqube.xml coverage.json summary_coverage.json coveralls.json
