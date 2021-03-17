#!/usr/bin/env python3
import spacy
import sys
import functools

NEGATION_WORDS = {'not', 'never', 'neither', 'none', 'nothing', 'unable'}
WHEN_TYPES = {'DATE', 'TIME', 'EVENT'}
WHERE_PREP_WORDS = {'in', 'on', 'at', 'from'}
WHERE_TYPES = {'FAC', 'ORG', 'GPE', 'LOC', 'EVENT'}
WHEN_WORDS = {'time', 'date', 'century', 'year', 'month', 'day'}
WHERE_WORDS = {'place', 'area', 'region', 'location', 'country', 'city'}
WHY_WORDS = {'because', 'since', 'due to', 'as a result'}

class qa():
    def __init__(self, ARTICLE_FILE, QUESTION_FILE):
        self.ARTICLE_FILE = ARTICLE_FILE
        self.QUESTION_FILE = QUESTION_FILE
        # self.nlp = spacy.load("en_core_web_sm")
        self.nlp = spacy.load("en_core_web_lg")

        ## import questions
        with open(QUESTION_FILE, 'r') as f:
            lines = f.readlines()
            self.questions = []
            for line in lines:
                self.questions.append({'text': line, 'nlp': self.nlp(line)})
        
        ## import article
        with open(ARTICLE_FILE, 'r') as f:
            self.article = f.read()
            self.doc = self.nlp(self.article)

    def classify(self):
        ## question type classifcation
        ## 4 types: yes/no, wh-determiner, wh-pronoun, wh-adverb
        for question in self.questions:
            if question['nlp'][0].pos_ == 'AUX' or question['nlp'][0].tag_ == 'MD':
                question['type'] = 'yes/no'
                question['question_word'] = question['nlp'][0]
                question['question_verb'] = [child.children for child in question['nlp'][0].children]
            else:
                for token in question['nlp']:
                    # print(token.text, token.tag_)
                    if token.tag_ == 'WDT':
                        question['type'] = 'wh-determiner'
                        question['question_word'] = token
                        break
                    elif token.tag_ == 'WP':
                        question['type'] = 'wh-pronoun'
                        if token.i != 0:
                            before = [word.text for word in question['nlp'][:token.i]]
                            after = [word.text for word in question['nlp'][token.i:-2]]
                            before_text = ' '.join(before)
                            after_text = ' '.join(after)
                            text = after_text + ' ' + before_text[0].lower() + before_text[1:] + '?'
                            question['text'] = text
                            question['nlp'] = self.nlp(text)
                            question['question_word'] = question['nlp'][0]
                            question['dep_look_for'] = question['nlp'][0].dep_
                            question['question_head'] = question['nlp'][0].head.lemma_
                        else:
                            question['question_word'] = token
                            question['dep_look_for'] = token.dep_
                            question['question_head'] = token.head.lemma_
                        break
                    elif token.tag_ == 'WP$':
                        question['type'] = 'wh-pronoun, possesive'
                        question['question_word'] = token
                        break
                    elif token.tag_ == 'WRB':
                        question['type'] = 'wh-adverb'
                        question['question_word'] = token
                        break
                if 'type' not in question:
                    question['type'] = 'unknown'
                    question['question_word'] = 'unknown'
        ## find root verb
        for question in self.questions:
            question['question_verb'] = self.find_key_verb(question['nlp'])
    
    def find_key_verb(self, sentence):

        ## recursive method to find verb in children
        def find_v(tok):
            if tok.pos_ == 'AUX' or tok.pos_ == 'VERB':
                return tok.lemma_
            for child in tok.children:
                res = find_v(child)
                if res != 'unknown':
                    return res
            return 'unknown'

        for token in sentence:
            if token.dep_ == 'ROOT': # start from root
                return find_v(token)
        return 'unknown'

    def fix_capitalization(self, sentence):
        sent = self.nlp(sentence)
        if sent[0].pos_ == 'PROPN' or sent[0].ent_type_ == 'NORP':
            return sent.text
        else:
            return "{}{}".format(sent.text[0].lower(), sent.text[1:])

    def search_top_sentences(self, sentence = None):
        ## identify the sentences in the article with most "noun chunks" that appeared in the question
        if sentence:
            ## find noun chunks in question
            noun_chunks = []
            for chunk in sentence['nlp'].noun_chunks:
                if sentence['question_word'].text not in chunk.text:
                    noun_chunks.append(chunk)
            sentence['noun_chunks'] = noun_chunks

            ## count occurence of noun chunks in each sentence
            sent_nc_matches = []
            for sent in self.doc.sents:
                nc_bow = [1 for i in range(len(noun_chunks)+1)]
                for i in range(len(noun_chunks)):
                    nc_bow[i] += sent.text.lower().count(noun_chunks[i].text.lower())

                # another dimension that match the Root verb
                key_verb = self.find_key_verb(sent)
                if sentence['question_verb'] and sentence['question_verb'] == key_verb:
                    nc_bow[-1] += 1

                nc_bow_val = functools.reduce(lambda a,b:a*b, nc_bow, 1)
                if nc_bow_val > 1:
                    sent_nc_matches.append((nc_bow_val, sent))
            sent_nc_matches.sort(reverse=True)

            sentence['top_5_sentences'] = sent_nc_matches[:5]
        else:
            for question in self.questions:
                ## find noun chunks in question
                noun_chunks = []
                for chunk in question['nlp'].noun_chunks:
                    if question['question_word'].text not in chunk.text:
                        noun_chunks.append(chunk)
                question['noun_chunks'] = noun_chunks

                ## count occurence of noun chunks in each sentence
                sent_nc_matches = []
                for sentence in self.doc.sents:
                    nc_bow = [1 for i in range(len(noun_chunks)+1)]
                    for i in range(len(noun_chunks)):
                        nc_bow[i] += sentence.text.lower().count(noun_chunks[i].text.lower())
                    
                    # another dimension that match the Root verb
                    key_verb = self.find_key_verb(sentence)
                    if question['question_verb'] and question['question_verb'] == key_verb:
                        nc_bow[-1] += 1

                    nc_bow_val = functools.reduce(lambda a,b:a*b, nc_bow, 1)
                    if nc_bow_val > 1:
                        sent_nc_matches.append((nc_bow_val, sentence))
                sent_nc_matches.sort(reverse=True)

                question['top_5_sentences'] = sent_nc_matches[:5]
    

    def generate_answers(self):

        self.classify()
        self.search_top_sentences()

        ## naive attempt to answer the question using the top sentence
        for question in self.questions:
            question['top_5_answers'] = []

            if question['type'] == 'wh-determiner':
                # re-write the question either as wh-pronoun or wh-adverb
                qw = question['question_word']
                obj = qw.head
                text = question['text'][0].lower() + question['text'][1:]
                if obj.text in WHEN_WORDS:
                    text = text.replace(qw.text.lower(), 'when', 1).replace(obj.text, '', 1)
                    question['type'] = 'wh-adverb'
                elif obj.text in WHERE_WORDS:
                    text = text.replace(qw.text.lower(), 'where', 1).replace(obj.text, '', 1)
                    question['type'] = 'wh-adverb'
                else:
                    text = text.replace(qw.text.lower(), 'what', 1).replace(obj.text, '', 1)
                    question['type'] = 'wh-pronoun'
                text = text.replace('  ', ' ')
                question['text'] = text
                question['nlp'] = self.nlp(text)

                ## re-classify the new question
                for token in question['nlp']:
                    if token.tag_ == 'WP':
                        if token.i != 0:
                            before = [word.text for word in question['nlp'][:token.i]]
                            after = [word.text for word in question['nlp'][token.i:-2]]
                            before_text = ' '.join(before)
                            after_text = ' '.join(after)
                            text = after_text + ' ' + before_text[0].lower() + before_text[1:] + '?'
                            question['text'] = text
                            question['nlp'] = self.nlp(text)
                            question['question_word'] = question['nlp'][0]
                            question['dep_look_for'] = question['nlp'][0].dep_
                            question['question_head'] = question['nlp'][0].head.lemma_
                        else:
                            question['question_word'] = token
                            question['dep_look_for'] = token.dep_
                            question['question_head'] = token.head.lemma_
                        break
                    elif token.tag_ == 'WRB':
                        question['question_word'] = token
                        break
                question['question_verb'] = self.find_key_verb(question['nlp'])
                self.search_top_sentences(question)

            if question['type'] == 'yes/no':
                question['top_5_answers'] = []
                for (num, sentence) in question['top_5_sentences']:
                    lemmas = [token.lemma_ for token in sentence]
                    found = False
                    for word in NEGATION_WORDS:
                        if word in lemmas:
                            question['top_5_answers'].append('no')
                            found = True
                            break
                    if not found:
                        question['top_5_answers'].append('yes')

            elif question['type'] == 'wh-pronoun':
                question['top_5_answers'] = []
                for (num, sentence) in question['top_5_sentences']:
                    added = False
                    for token in sentence:
                        ## the token is what we are looking for if it has
                        ## the same dependency and head as the question word (what/who)
                        # print(token, token.dep_, token.head.lemma_, question['dep_look_for'], question['question_head'], added)
                        if (token.dep_ == question['dep_look_for']) and token.head.lemma_ == question['question_head'] and not added:
                            ## find the noun chunk for the token
                            token_in_chunk = False
                            for chunk in sentence.noun_chunks:
                                if token.text in chunk.text:
                                    question['top_5_answers'].append(self.fix_capitalization(chunk.text))
                                    token_in_chunk = True
                                    break
                            if not token_in_chunk:
                                question['top_5_answers'].append(self.fix_capitalization(token.text))
                            added = True
                        # break

                    if not added:
                        question['top_5_answers'].append(sentence.text.strip())

            elif question['type'] == 'wh-pronoun, possesive':
                # DONE: answer "Whose" questions
                question['top_5_answers'] = []
                for (num, sentence) in question['top_5_sentences']:
                    added = False
                    possesion = question['question_word'].head
                    for token in sentence:
                        if token.dep_ == 'poss' and token.head.lemma_ == possesion.lemma_:
                            sub_phrase = self.doc[token.head.left_edge.i : token.head.right_edge.i + 1]
                            question['top_5_answers'].append(self.fix_capitalization(sub_phrase.text))
                            added = True
                            break
                    if not added:
                        question['top_5_answers'].append(sentence.text.strip())

            elif question['type'] == 'wh-adverb':
                question['top_5_answers'] = []

                if question['question_word'].lemma_ == 'when':
                    # DONE: answer "When" questions
                    for (num, sentence) in question['top_5_sentences']:
                        added = False
                        for token in sentence:

                            ## traverse through the dependency tree of from the ROOT
                            ## if the child (or grandchild) is DATE or TIME
                            ## find the named entity it is part of

                            if token.head.lemma_ == question['question_verb'] and not added: 

                                def find_time(word_tok):
                                    if word_tok.ent_type_ in WHEN_TYPES:
                                        return word_tok
                                    else:
                                        for child in word_tok.children:
                                            return find_time(child)

                                token_found = find_time(token)
                                if token_found:
                                    # print(token_found.text)
                                    for ent in sentence.ents:
                                        if ent.label_ == token_found.ent_type_ and ent.start_char <= token_found.idx and ent.end_char >= token_found.idx:
                                            question['top_5_answers'].append(self.fix_capitalization(ent.text))
                                            added = True
                                            break
                        if not added:

                            ## if not found, try to find any DATE/TIME in named entities

                            for ent in sentence.ents:
                                if ent.label_ == 'DATE' or ent.label_ == 'TIME':
                                    question['top_5_answers'].append(self.fix_capitalization(ent.text))
                                    added = True
                                    break
                            if not added:
                                question['top_5_answers'].append(sentence.text.strip())

                elif question['question_word'].lemma_ == 'where':
                    # DONE: answer "Where" questions
                    for (num, sentence) in question['top_5_sentences']:
                        ## find noun chunks as prepositions with in/at/on/etc that did not appear in the question.
                        added = False
                        potential_chunks = []
                        potential_answer = sentence.text.strip()
                        for noun_chunk in sentence.noun_chunks:
                            if noun_chunk.text not in question['text'] and noun_chunk.root.head.text in WHERE_PREP_WORDS:
                                potential_chunks.append(noun_chunk)
                        ## prioritize NE for places in prepositions with in/at/on
                        for noun_chunk in potential_chunks:
                            for token in noun_chunk:
                                if token.ent_type_ in WHERE_TYPES and not added:
                                    for ent in sentence.ents:
                                        if ent.label_ == token.ent_type_ and ent.start_char <= token.idx and ent.end_char >= token.idx:
                                            potential_answer = self.fix_capitalization(ent.text)
                                            added = True
                                            # print(ent.text, ent.label_, 'first')
                                            break
                        ## if not found, go for noun chunks in preposition with in/at/on
                        if not added:
                            for noun_chunk in potential_chunks:
                                potential_answer = self.fix_capitalization(noun_chunk.text)
                                added = True
                                # print(noun_chunk.text, noun_chunk.root.head.text, 'any chunk')
                                break
                        ## finally, go for any NE for places
                        if not added:
                            for ent in sentence.ents:
                                if ent.label_ in WHERE_TYPES:
                                    potential_answer = self.fix_capitalization(ent.text)
                                    added = True
                                    # print(ent.text, ent.label_, 'any NE')
                                    break
                        question['top_5_answers'].append(potential_answer)

                elif question['question_word'].lemma_ == 'why':
                    # DONE: answer "Why" questions
                    for (num, sentence) in question['top_5_sentences']:
                        added = False
                        for token in sentence:
                            if token.pos_ == 'SCONJ':
                                sub_phrase = self.doc[token.left_edge.i : token.right_edge.i + 1]
                                if len(sub_phrase) > 1:
                                    sub_phrase = self.fix_capitalization(sub_phrase.text)
                                    question['top_5_answers'].append(sub_phrase)
                                else:
                                    question['top_5_answers'].append(sentence.text.strip())
                                added = True
                                break
                        if not added:
                            question['top_5_answers'].append(sentence.text.strip())
                        
                else:
                    question['top_5_answers'] = [sentence.text.strip() for (_, sentence) in question['top_5_sentences']]

            else:
                question['top_5_answers'] = [sentence.text.strip() for (_, sentence) in question['top_5_sentences']]

    def answer(self):
        self.generate_answers()
        answers = []
        for question in self.questions:
            if 'top_5_answers' in question and question['top_5_answers']:
                for i in range(len(question['top_5_answers'])):
                    if question['top_5_answers'][i] != 'unknown':
                        answers.append(question['top_5_answers'][i])
                        break
                    elif i == len(question['top_5_answers']) - 1:
                        answers.append('unknown')
            else:
                answers.append('unknown')
        print('\n'.join(answers))

    def __str__(self):
        string = ""
        for question in self.questions:
            string += '''question: {}
type: {}
question_word: {}
question_verb: {}
noun_chunks: {}
top_5_sentences: {}
top_5_answers: {}\n\n'''.format(question['text'], question['type'],
                question['question_word'], question['question_verb'],
                question['noun_chunks'], question['top_5_sentences'], question['top_5_answers'])
        return string
if __name__ == '__main__':

    ARTICLE_FILE = sys.argv[1]
    QUESTION_FILE = sys.argv[2]

    answerer = qa(ARTICLE_FILE, QUESTION_FILE)
    answerer.answer()
    # print(answerer)



