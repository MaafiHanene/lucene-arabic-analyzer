# lucene-arabic-analyzer
Apache Lucene analyzer for Arabic language with root based stemmer.

Stemming algorithms are used in information retrieval systems, text classifiers, indexers and text mining to extract roots of different words, so that words derived from the same stem or root are grouped together. Many stemming algorithms were built in different natural languages.
This implementation is based on [Khoja stemmer](http://zeus.cs.pacificu.edu/shereen/research.htm#stemming) which is one of the widely used Arabic stemmers.

`ArabicRootExtractorAnalyzer` is responsible to do the following:

1. Normalize input text by removing diacritics: e.g. "الْعَالَمِينَ" will be converted to "العالمين".
2. Extract word's root: e.g. "العالمين" will be converted to "علم".

This way, documents will be indexed depending on its words roots, so, when you want to search in the index, you can input "علم" or "عالم" to get all documents containing "الْعَالَمِينَ".

## Installation

**Maven**
```xml
<dependency>
	<groupId>com.github.msarhan</groupId>
	<artifactId>lucene-arabic-analyzer</artifactId>
	<version>1.1.0</version>
</dependency>
```
**Gradle**
```gradle
repositories {
	mavenCentral()
}
dependencies {
	compile group: 'com.github.msarhan', name: 'lucene-arabic-analyzer', version:'1.1.0'
}
```

## Usage

```java
//Initialize the index
Directory index = new RAMDirectory();
Analyzer analyzer = new ArabicRootExtractorAnalyzer();
IndexWriterConfig config = new IndexWriterConfig(analyzer);
IndexWriter writer = new IndexWriter(index, config);

Document doc = new Document();
doc.add(new StringField("number", "1", Field.Store.YES));
doc.add(new TextField("title", "بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيمِ", Field.Store.YES));
writer.addDocument(doc);

doc = new Document();
doc.add(new StringField("number", "2", Field.Store.YES));
doc.add(new TextField("title", "الْحَمْدُ لِلَّهِ رَبِّ الْعَالَمِينَ", Field.Store.YES));
writer.addDocument(doc);

doc = new Document();
doc.add(new StringField("number", "3", Field.Store.YES));
doc.add(new TextField("title", "الرَّحْمَنِ الرَّحِيمِ", Field.Store.YES));
writer.addDocument(doc);
writer.close();
//~

//Query the index
String queryStr = "راحم";
Query query = new QueryParser("title", analyzer)
		.parse(queryStr);

int hitsPerPage = 5;
IndexReader reader = DirectoryReader.open(index);
IndexSearcher searcher = new IndexSearcher(reader);
TopDocs docs = searcher.search(query, hitsPerPage, Sort.INDEXORDER);

ScoreDoc[] hits = docs.scoreDocs;
//~

//Print results
System.out.println("Found " + hits.length + " hits:");
for (ScoreDoc hit : hits) {
	int docId = hit.doc;
	Document d = searcher.doc(docId);
	System.out.printf("\t(%s): %s\n", d.get("number"), d.get("title"));
}
/*
	This will print:
	Found 2 hits:
		(1): بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيمِ
		(3): الرَّحْمَنِ الرَّحِيمِ
 */
//~
```

### Usage of `ArabicRootExtractorStemmer`
```java
ArabicRootExtractorStemmer stemmer = new ArabicRootExtractorStemmer();

assertEquals("رحم", stemmer.stem("الرَّحْمَنِ"));
assertEquals("رحم", stemmer.stem("الرَّحِيمِ"));
assertEquals("علم", stemmer.stem("الْعَالَمِينَ"));
assertEquals("عبد", stemmer.stem("نَعْبُدُ"));
assertEquals("أمن", stemmer.stem("الْمُؤْمِنِينَ"));
assertEquals("صلح", stemmer.stem("الصَّالِحَاتِ"));
assertEquals("فلح", stemmer.stem("تُفْلِحُوا"));
assertEquals("نزع", stemmer.stem("يَتَنَازَعُونَ"));
assertEquals("شرب", stemmer.stem("الشَّرَابُ"));
assertEquals("غفل", stemmer.stem("أَغْفَلْنَا"));
assertEquals("قلب", stemmer.stem("مُنقَلَبًا"));
assertEquals("بقي", stemmer.stem("وَالْبَاقِيَاتُ"));
assertEquals("جرم", stemmer.stem("الْمُجْرِمِينَ"));
assertEquals("ظلم", stemmer.stem("لِلظَّالِمِينَ"));
```
