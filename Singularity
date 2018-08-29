FROM ubuntu:17.10

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    cpanminus \
    libfile-nfslock-perl \
    libperlio-gzip-perl \
    libtest-deep-perl \
    libtest-utf8-perl \
    libbio-perl-perl \
    hmmer \
    wget \
    libboost-iostreams-dev \
    zlib1g-dev \
    libgsl-dev \
    libsqlite3-dev \
    libboost-graph-dev \
    libsuitesparse-dev \
    liblpsolve55-dev \
    bamtools \
    libbamtools-dev \
    nano \
    gcc-5 \
    g++-5 \
    libglib2.0-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN  ["cpanm", "Error", "Error::Simple", "File::Which", "Inline", "Perl::Unsafe::Signals", "Proc::ProcessTable", "URI::Escape", "Bit::Vector", "Inline::C", "forks", "forks::shared", "IO::All", "DBD::SQLite", "IO::Prompt", "Text::Soundex"]

RUN wget http://bioinf.uni-greifswald.de/augustus/binaries/augustus-3.3.1.tar.gz \
    && tar -xvf augustus*.tar.gz \
    && rm augustus*.tar.gz \
    && cd augustus*/src \
    && make

RUN wget http://ftp.ebi.ac.uk/pub/software/vertebrategenomics/exonerate/exonerate-2.2.0-x86_64.tar.gz \
    && tar -xzf exonerate-2.2.0-x86_64.tar.gz \
    && rm exonerate-2.2.0-x86_64.tar.gz

RUN wget ftp://ftp.ncbi.nlm.nih.gov/blast/executables/rmblast/2.2.28/ncbi-rmblastn-2.2.28-x64-linux.tar.gz \
    && tar -xzf ncbi-rmblastn-2.2.28-x64-linux.tar.gz \
    && rm ncbi-rmblastn-2.2.28-x64-linux.tar.gz

RUN wget ftp://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/2.7.1/ncbi-blast-2.7.1+-x64-linux.tar.gz \
    && tar -xzf ncbi-blast-2.7.1+-x64-linux.tar.gz \
    && rm ncbi-blast-2.7.1+-x64-linux.tar.gz \
    && cp -r ncbi-blast-2.7.1+/bin/* ncbi-rmblastn-2.2.28/bin

COPY trf /RepeatMasker/

RUN wget http://www.repeatmasker.org/RepeatMasker-open-4-0-6.tar.gz \
    && tar -xzf RepeatMasker-open*.tar.gz \
    && rm -f RepeatMasker-open*.tar.gz \
    && cd RepeatMasker \
    && sed -e 's/\/usr\/local\/hmmer/\/usr\/bin/g;' \
        -e 's/\/usr\/local\/rmblast/\/ncbi-rmblastn-2.2.28\/bin/g;' \
        -e 's/DEFAULT_SEARCH_ENGINE = "crossmatch"/DEFAULT_SEARCH_ENGINE = "ncbi"/g;' \
        -e 's/TRF_PRGM = ""/TRF_PRGM = "\/RepeatMasker\/trf"/g;' \
        RepeatMaskerConfig.tmpl > RepeatMaskerConfig.pm

COPY Libraries /RepeatMasker/Libraries

RUN wget http://www.mpich.org/static/downloads/3.2.1/mpich-3.2.1.tar.gz \
    && tar -xzf mpich-3.2.1.tar.gz \
    && rm mpich-3.2.1.tar.gz \
    && cd mpich-3.2.1 \
    && ./configure --disable-fortran --enable-sharedlibs \
    && make \
    && make test \
    && make install

RUN update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-5 60 --slave /usr/bin/g++ g++ /usr/bin/g++-5

RUN wget --no-check-certificate https://launchpad.net/ubuntu/+archive/primary/+files/snap_2013-11-29.orig.tar.gz \
    && tar -xzf snap_2013-11-29.orig.tar.gz \
    && rm snap_2013-11-29.orig.tar.gz \
    && cd snap \
    && make

ENV ZOE="/snap/Zoe"

ENV PATH="$PATH:/exonerate-2.2.0-x86_64/bin:/ncbi-rmblastn-2.2.28/bin:/maker/bin:/RepeatMasker:/snap:/maker/bin:/home/projects/src:"

RUN wget http://yandell.topaz.genetics.utah.edu/maker_downloads/EDD0/9498/2D2F/195FF7F5C137C2ADB96B8F1F1EEB/maker-2.31.9.tgz \
    && tar -xzf maker-2.31.9.tgz \
    && rm maker-2.31.9.tgz \
    && cd maker/src \
    && echo y| perl Build.PL \
    && ./Build install

COPY data /home/projects/data
COPY ctl /home/projects/ctl
COPY src /home/projects/src

RUN cd /RepeatMasker \
	&& perl -i -0pe 's/^#\!.*perl.*/#\!\/usr\/bin\/env perl/g' \
	RepeatMasker \
    DateRepeats \
    ProcessRepeats \
    RepeatProteinMask \
    DupMasker \
    util/queryRepeatDatabase.pl \
    util/queryTaxonomyDatabase.pl \
    util/rmOutToGFF3.pl \
    util/rmToUCSCTables.pl

RUN cd RepeatMasker \
    && /ncbi-rmblastn-2.2.28/bin/makeblastdb -dbtype nucl -in Libraries/RepeatMasker.lib > /dev/null 2>&1 \
    && /ncbi-rmblastn-2.2.28/bin/makeblastdb -dbtype prot -in Libraries/RepeatPeps.lib > /dev/null 2>&1

