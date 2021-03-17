#!/usr/bin/env python3
import spacy
import inflect
import re
import sys

WHEN_TYPES = {'DATE', 'TIME'}
WHERE_TYPES = {'FAC', 'GPE', 'LOC'}

class q_generator():
	def __init__(self, ARTICLE_FILE, NUM_QUES):
		self.ARTICLE_FILE = ARTICLE_FILE
		self.p = inflect.engine()
		self.NUM_QUES = NUM_QUES

		self.locPre = ['at', 'in', 'to']
		self.ignore = ['he', 'him', 'his', 'himself', 'she', 'her', 'hers', 'herself', 'they', 'them', 'their', 'theirs',
			'themselves', 'this', 'these', 'those', 'i', 'me', 'my', 'mine', 'myself', 'it', 'its', 'itself',
			'you', 'your', 'yours', 'yourself']
		self.sentences = list()
		self.who = list()
		self.what = list()
		self.when = list()
		self.where = list()
		self.why = list()

	def readInput(self):
		with open(ARTICLE_FILE, 'r') as f:
			txt = f.read()
			self.sentences =[sent.text for sent in list(nlp(txt).sents)]

	def clear_multiple_line(self):
		f = lambda que: len(que.split('\n')) == 1
		self.who = list(filter(f, self.who))
		self.what = list(filter(f, self.what))
		self.when = list(filter(f, self.when))
		self.where = list(filter(f, self.where))
		self.why = list(filter(f, self.why))


	def clear_pronoun(self):
		def f(que): 
			for token in que.split():
				if token.lower() in self.ignore:
					return False
			return True
		self.who = list(filter(f, self.who))
		self.what = list(filter(f, self.what))
		self.when = list(filter(f, self.when))
		self.where = list(filter(f, self.where))
		self.why = list(filter(f, self.why))

	def clear_redundent(self):
		f = lambda que: len(que.split(',')) <= 3
		self.who = list(filter(f, self.who))
		self.what = list(filter(f, self.what))
		self.when = list(filter(f, self.when))
		self.where = list(filter(f, self.where))
		self.why = list(filter(f, self.why))

	def clean_mess(self):
		f = lambda que: str(que + '"') if que.count('"') % 2 == 1 else que
		self.who = [f(que) for que in self.who]
		self.what = [f(que) for que in self.what]
		self.when = [f(que) for que in self.when]
		self.where = [f(que) for que in self.where]
		self.why = [f(que) for que in self.why]

		f = lambda que: '(' not in que and ')' not in que
		self.who = list(filter(f, self.who))
		self.what = list(filter(f, self.what))
		self.when = list(filter(f, self.when))
		self.where = list(filter(f, self.where))
		self.why = list(filter(f, self.why))

		f = lambda que: que.replace('  ', ' ').replace(' , ', ', ')
		self.who = [f(que) for que in self.who]
		self.what = [f(que) for que in self.what]
		self.when = [f(que) for que in self.when]
		self.where = [f(que) for que in self.where]
		self.why = [f(que) for que in self.why]

		f = lambda que: len(set(que.replace('the', '').split())) == len(que.replace('the', '').split())
		self.who = list(filter(f, self.who))
		self.what = list(filter(f, self.what))
		self.when = list(filter(f, self.when))
		self.where = list(filter(f, self.where))
		self.why = list(filter(f, self.why))

		self.who = [self.clean_end(que) for que in self.who]
		self.what = [self.clean_end(que) for que in self.what]
		self.when = [self.clean_end(que) for que in self.when]
		self.where = [self.clean_end(que) for que in self.where]
		self.why = [self.clean_end(que) for que in self.why]


	def clear_short(self):
		f = lambda que: len(que.split(' ')) >= 8
		self.who = list(filter(f, self.who))
		self.what = list(filter(f, self.what))
		self.when = list(filter(f, self.when))
		self.where = list(filter(f, self.where))
		self.why = list(filter(f, self.why))
		
	def sort(self):
		self.who.sort(key=len)
		# self.who.reverse()
		self.what.sort(key=len)
		# self.what.reverse()
		self.where.sort(key=len)
		# self.where.reverse()
		self.when.sort(key=len)
		# self.when.reverse()
		self.why.sort(key=len)
		# self.why.reverse()

	def rank(self):
		self.clear_redundent()
		self.clear_multiple_line()
		self.clear_pronoun()
		self.clean_mess()
		self.clear_short()
		self.sort()

	def clean_end(self, line):
		segments = line.split('?')[0].split(',')
		segs_param = [nlp(seg) for seg in segments]
		segs_param = segs_param if len(segs_param[-1]) > 2 else segs_param[:-1]
		segments = [seg.text for seg in segs_param]
		text = ','.join(segments) + '?'
		return text
	

	def generate_quesions(self):
		for line in self.sentences:
			line = self.clean_line(line)
			self.generate_question_perLine(line)
		self.rank()

		out = list()
		maxLen = max(len(self.who), len(self.what),len(self.when), len(self.where),len(self.why))
		for i in range(maxLen):
			if i < len(self.who):
				out.append(self.who[i])
			if i < len(self.what):
				out.append(self.what[i])
			if i < len(self.when):
				out.append(self.when[i])
			if i < len(self.where):
				out.append(self.where[i])
			if i < len(self.why):
				out.append(self.why[i])
		outPutLen = min(self.NUM_QUES, len(out))
		for i in range(outPutLen):
			print(out[i])

	def clean_line(self, line):
		idx = len(line)-1
		while idx >= 0 and not line[idx].isalpha() and not line[idx].isdigit():
			idx -= 1
		return line[:idx+1]

	def generate_question_perLine(self, line):
		parameter = nlp(line)
		self.who_what_questions(line, parameter)
		self.when_questions(line, parameter)
		self.why_questions(line, parameter)
		self.where_questions(line, parameter)

	def when_questions(self, line, parameter):
		noun_phrase = [chunk for chunk in parameter.noun_chunks]
		verbs = [token for token in parameter if token.pos_ == "VERB"]
		if not noun_phrase or not verbs:
			return
		ents = [ent for ent in parameter.ents]
		ent_labels = [ent.label_ for ent in ents]
		time_ents = list(filter(lambda ent : ent.label_ in WHEN_TYPES, ents))
		if len(time_ents) == 0:
			return
		else:
			found = False
			for ent in time_ents:
				for token in parameter:
					if ent.label_ == token.ent_type_ and ent.start_char <= token.idx and ent.end_char >= token.idx:
						if token.dep_ == 'pobj' and token.head.lemma_ in ['at', 'in', 'on']:
							prep_idx = token.head.idx
							line = (line[:prep_idx].strip() + ' ' + line[ent.end_char:].strip()).strip()
							found = True
							break
		if not found:
			return

		if noun_phrase[0].start == 0 and verbs[0].i == noun_phrase[0].end and noun_phrase[0].text.lower() not in self.ignore:
			delimiter = "When"
			exist_extra_words = len(parameter) > noun_phrase[0].end
			if parameter[noun_phrase[0].start].pos_ == 'PROPN' or parameter[noun_phrase[0].start].ent_type_ == 'NORP':
				target = noun_phrase[0].text
			else:
				target = "{}{}".format(noun_phrase[0].text[0].lower(), noun_phrase[0].text[1:])
			tense_word = self.findCorrectTense(verbs[0].tag_)
			if exist_extra_words and line[parameter[noun_phrase[0].end + 1].idx:] != '':
				ques = "{} {} {} {} {}?".format(delimiter, tense_word, target, verbs[0].lemma_, line[parameter[noun_phrase[0].end + 1].idx:])
			else:
				ques = "{} {} {} {}?".format(delimiter, tense_word, target, verbs[0].lemma_)
			self.when.append(ques)


	def where_questions(self, line, parameter):
		loc = -1
		proper_sent = False
		for entity in parameter.ents:
			if entity.label_ in WHERE_TYPES:
				loc = entity.start_char
				for chunk in parameter.noun_chunks:
					if parameter[chunk.start].idx == loc and chunk.root.head.text.lower() in self.locPre:
						proper_sent = True
						break
				if proper_sent:
					break
		if proper_sent:
			verb = None
			aux = False
			for i, token in enumerate(parameter):
				if token.dep_ == 'ROOT':
					if token.text in self.ignore or (i > 0 and parameter[i-1].dep_ == 'aux'):
						aux = True
					else:
						verb = token
					break
			noun = None
			if verb:
				for chunk in parameter.noun_chunks:
					if chunk.root.head and chunk.root.head.text == verb.text:
						noun = chunk
						break
				if noun and not aux:
					append = line[verb.idx + len(verb) + 1: loc-4]
					if len(append) > 0:
						tense_word = self.findCorrectTense(verb.tag_)
						if noun[0].pos_ == 'PROPN' or noun[0].ent_type_ == 'NORP':
							target = noun.text
						else:
							target = "{}{}".format(noun.text[0].lower(), noun.text[1:])
						self.where.append("Where {} {} {} {}?".format(tense_word, target, verb.lemma_, append))

	def why_questions(self, line, parameter):
		for i, token in enumerate(parameter):
			if (token.text == 'because' and parameter[i+1].text != 'of') or (token.text == 'due' and parameter[i+1].text == 'to') and i > 3:
				right_edge_delete = token.head.left_edge.idx - 1
				right_edge_delete = token.idx-1
				line = line[:right_edge_delete]
				if parameter[0].pos_ != 'PROPN' and parameter[0].ent_type_ != 'NORP' and len(line):
					self.why.append("Why is that {}{}?".format(line[0].lower(), line[1:]))
				else:
					self.why.append("Why is that {}?".format(line))
				break

	def who_what_questions(self, line, parameter):
		questions = list()
		noun_phrase = [chunk for chunk in parameter.noun_chunks]
		verbs = [token for token in parameter if token.pos_ == "VERB"]
		cur_line_named_entity = parameter.ents
		if len(noun_phrase) < 1  or  len(verbs) < 1:
			return questions

		#pattern 1 who/what verbs
		if noun_phrase[0].start == 0 and verbs[0].i == noun_phrase[0].end:
			delimiter = "What"
			if self.ifHuman(parameter, 0):
				delimiter = "Who"
			if delimiter == "Who":
				self.who.append("{} {}?".format(delimiter, line[parameter[noun_phrase[0].end].idx:]))
			else:
				self.what.append("{} {}?".format(delimiter, line[parameter[noun_phrase[0].end].idx:]))

		if len(noun_phrase) < 2:
			return questions
		#pattern 2 noun verbs non_chuck -> who/what did noun verb
		if noun_phrase[0].start == 0 and verbs[0].i == noun_phrase[0].end and verbs[0].i + 1 == noun_phrase[1].start and noun_phrase[0].text.lower() not in self.ignore:
			delimiter = "What"
			exist_extra_words = len(parameter) > noun_phrase[1].end
			if self.ifHuman(parameter, 1):
				delimiter = "Who"
			target = noun_phrase[0].text
			if not self.ifHuman(parameter, 0):
				if parameter[noun_phrase[0].start].pos_ == 'PROPN' or parameter[noun_phrase[0].start].ent_type_ == 'NORP':
					target = noun_phrase[0].text
				else:
					target = "{}{}".format(noun_phrase[0].text[0].lower(), noun_phrase[0].text[1:])
			tense_word = self.findCorrectTense(verbs[0].tag_)
			if delimiter == "Who":
				if exist_extra_words:
					self.who.append("{} {} {} {} {}?".format(delimiter, tense_word, target, verbs[0].lemma_, line[parameter[noun_phrase[1].end].idx:]))
				else:
					self.who.append("{} {} {} {}?".format(delimiter, tense_word, target, verbs[0].lemma_))
			else:
				if exist_extra_words:
					self.what.append("{} {} {} {} {}?".format(delimiter, tense_word, target, verbs[0].lemma_, line[parameter[noun_phrase[1].end].idx:]))
				else:
					self.what.append("{} {} {} {}?".format(delimiter, tense_word, target, verbs[0].lemma_))

	def ifHuman(self, parameter, idx_chunk):
		tokens = [chunk for chunk in parameter.noun_chunks][idx_chunk]
		tokens_text = [token.text for token in tokens]
		tags = [token.tag_ for token in tokens]
		for ent in parameter.ents:
			if ent.text in tokens.text and ent.label_ != 'PERSON':
				return False
		if len(tags) == 1:
			if (tags[0] == 'PRP' and tokens[0].text.lower() != 'it') or tags[0] == "NNP" or tags[0] == "NNPS":
				return True
			return False
		else:
			for tag in tags:
				if tag != 'NNP':
					return False
			return True
	
	def findCorrectTense(self, tag):
		if tag == 'VBD' or tag == 'VBN':
			return 'did'
		elif tag == 'VBZ':
			return 'does'
		else:
			return 'do'



if __name__ == '__main__':
	nlp = spacy.load("en_core_web_lg")

	ARTICLE_FILE = sys.argv[1]
	NUM_QUES = sys.argv[2]

	generator = q_generator(ARTICLE_FILE, int(NUM_QUES))
	generator.readInput()
	generator.generate_quesions()



	