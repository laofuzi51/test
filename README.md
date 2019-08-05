
import re
from collections import defaultdict
class BayesWordTransfer(object):
    """
    贝叶斯拼写检查器
        分析：
            P(c), 文章中出现一个正确拼写词 c 的概率, 也就是说, 在英语文章中, c 出现的概率有多大
            P(w|c), 在用户想键入 c 的情况下敲成 w 的概率. 因为这个是代表用户会以多大的概率把 c 敲错成 w
            argmaxc, 用来枚举所有可能的 c 并且选取概率最大的
    """

    def __init__(self):
        self.input_word = input("请输入单词: ")
        self.alphabet = 'abcdefghijklmnopqrstuvwxyz'

    def read_file(self, path):
        """读取语料库"""
        with open(path, "r") as f:
            return f.read()

    def word_handle(self, text):
        """抽取语料库中的单词"""
        return re.findall(r'[a-z]+', text.lower())

    def train(self, features):
        """统计词库：先验概率，计算词频"""
        model = defaultdict(lambda: 1)  # 防止先验概率为0，统计未出现的单词设默认值为1
        for w in features:
            model[w] += 1
        return model

    # 计算错误单词发生的概率
    def known(self, words):
        """方式1：编辑距离为0，判断用户输入的单词是否在词库中"""
        return set(w for w in words if w in self.new_words)

    def edits1(self, words):
        """方式2：计算编辑距离为1"""
        n = len(words)

        distance01 = [words[0:i] + words[i + 1:] for i in range(n)]  # 删除下标为i的元素
        distance02 = [words[0:i] + words[i + 1] + words[i] + words[i + 2:] for i in
                      range(n - 1)]  # 下标为i的元素和下标为i+1的元素交换位置
        distance03 = [words[0:i] + c + words[i + 1:] for i in range(n) for c in self.alphabet]  # 下标为i的元素分别替换为a-z的字母
        distance04 = [words[0:i] + c + words[i:] for i in range(n + 1) for c in self.alphabet]  # 在最前面插入a-z的字母

        return set(distance01 + distance02 + distance03 + distance04)

    def known_edits2(self, words):
        """方式3：计算编辑距离为2，并判断是否在词库中"""
        return set(w2 for w1 in words for w2 in w1 if w2 in self.new_words)

    def correct(self, words):
        """计算在词库中匹配正确的单词：类似贝叶斯公式"""
        candidates = self.known([words]) or self.known(self.edits1(words)) or self.known_edits2(words) or [words]
        return max(candidates, key=lambda x: self.new_words[x])

    def main(self):
        """逻辑处理"""
        # 1、读取文件
        text = self.read_file("./big.txt")

        # 2、处理文字
        words = self.word_handle(text)

        # 3、统计文字：先验概率
        self.new_words = self.train(words)

        # 4、由贝叶斯计算出匹配结果
        print(self.correct(self.input_word))


if __name__ == '__main__':
    tool = BayesWordTransfer()
    tool.main()
