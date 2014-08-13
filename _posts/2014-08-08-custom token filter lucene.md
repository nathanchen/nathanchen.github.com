---
layout: post
title: "Custom Token Filter Lucene"
category: 
tags: ["Lucene"]
---
{% include JB/setup %}

In order to implement our filter we will extends the `TokenFilter` class from the `org.apache.lucene.analysis` and we will override the `incrementToken` method. This method returns a `boolean` value: **if a value is still available for processing in the token stream, this method should return true**, is the token in the token stream shouldn't be further analyzed this method should return false.

	package pl.solr.analysis;
	
	import java.io.IOException;
	
	import org.apache.lucene.analysis.TokenFilter;
	import org.apache.lucene.analysis.TokenStream;
	import org.apache.lucene.analysis.tokenattributes.CharTermAttribute;
	
	public final class ReverseFilter extends TokenFilter {
	  private CharTermAttribute charTermAttr;
	
	  protected ReverseFilter(TokenStream ts) {
	    super(ts);
	    this.charTermAttr = addAttribute(CharTermAttribute.class);
	  }
	
	  @Override
	  public boolean incrementToken() throws IOException {
	    if (!input.incrementToken()) {
	      return false;
	    }
	
	    int length = charTermAttr.length();
	    char[] buffer = charTermAttr.buffer();
	    char[] newBuffer = new char[length];
	    for (int i = 0; i < length; i++) {
	      newBuffer[i] = buffer[length - 1 - i];
	    }
	    charTermAttr.setEmpty();
	    charTermAttr.copyBuffer(newBuffer, 0, newBuffer.length);
	    return true;
	  }
	}
	
A `TokenStream` is a class that can produce a series of tokens when requested, but there are two different styles of `TokenStreams`: `Tokenizer` and `TokenFilter`.

A `Tokenizer` reads characters from a `java.io.Reader` and creates tokens, whereas a `TokenFilter` takes tokens in, and produces new tokens by either adding or removing whole tokens or altering the attributes of the incoming tokens.

	public TokenStream tokenStream(String fieldName, Reader reader)
	{
		return new StopFilter(true, new LowerCaseTokenizer(reader), stopWords);
	}
	
In this anlyzer, **`LowerCaseTokenizer` produces the initial set of tokens from a Reader and feeds them to a StopFilter**. The `LowerCaseTokenizer` emits tokens that are adjacent letters in the original text, lowercasing each of the characters in the process. Following this word tokenizer and lowercasing, **`StopFilter` removes words in a stop-word list while preserving accurate `positionIncrements`**.

Buffering is a feature that's commonly needed in the `TokenStream` implementations. Low-level `Tokenizers` do this to buffer up characters to form tokens at boundaries such as whitespace or nonletter characters. `TokenFilters` that emit additional tokens into the stream they're filtering must queue an incoming token and the additional ones and emit them one at a time.

the `TokenStream` never explicitly creates a single object holding all attributes for the token. Instead, you interact with a separate reused attribute interface for each element of the token.

`TokenStream` subclasses from a class called `AttributeSource`. `AttributeSource` is a useful and generic means of providing strongly typed yet fully extensible attributes without reuquiring runtime casting, thus resulting in good performance.