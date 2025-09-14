package main

import (
	"bufio"
	"encoding/json"
	"flag"
	"fmt"
	"os"
	"strings"
)

type Question struct {
	Q       string   `json:"q"`
	Choices []string `json:"choices"`
	A       int      `json:"a"` // index of correct choice
}

func loadQuestions(path string) ([]Question, error) {
	// if path empty or not found, return built-in sample
	if path == "" {
		return sampleQuestions(), nil
	}
	b, err := os.ReadFile(path)
	if err != nil {
		return nil, err
	}
	var qs []Question
	if err := json.Unmarshal(b, &qs); err != nil {
		return nil, err
	}
	return qs, nil
}

func sampleQuestions() []Question {
	return []Question{
		{Q: "What language is known for 'goroutines'?", Choices: []string{"Python", "Go", "Ruby", "PHP"}, A: 1},
		{Q: "Which planet is known as the Red Planet?", Choices: []string{"Venus", "Mars", "Jupiter", "Saturn"}, A: 1},
		{Q: "What does HTML stand for?", Choices: []string{"Hyperlinks Text Markup Language", "HyperText Markup Language", "HighText Machine Language", "HyperText Markdown Language"}, A: 1},
	}
}

func main() {
	file := flag.String("q", "", "path to questions JSON (optional). If omitted builtin questions are used.")
	shuffle := flag.Bool("shuffle", false, "shuffle questions (not implemented; placeholder)")
	flag.Parse()

	qs, err := loadQuestions(*file)
	if err != nil {
		fmt.Println("Failed to load questions:", err)
		return
	}
	if len(qs) == 0 {
		fmt.Println("No questions available.")
		return
	}

	reader := bufio.NewReader(os.Stdin)
	score := 0

	fmt.Println("Trivia — answer by typing the number of the choice and pressing Enter.")
	fmt.Println("Type 'q' to quit any time.\n")

	for i, q := range qs {
		fmt.Printf("Q%d: %s\n", i+1, q.Q)
		for j, c := range q.Choices {
			fmt.Printf("  %d) %s\n", j+1, c)
		}
		for {
			fmt.Print("> ")
			text, _ := reader.ReadString('\n')
			text = strings.TrimSpace(text)
			if text == "q" {
				fmt.Println("Quitting early.")
				fmt.Printf("Score: %d/%d\n", score, i)
				return
			}
			var ans int
			_, err := fmt.Sscan(text, &ans)
			if err != nil || ans < 1 || ans > len(q.Choices) {
				fmt.Println("Please type a valid choice number.")
				continue
			}
			if ans-1 == q.A {
				fmt.Println("Correct!")
				score++
			} else {
				fmt.Printf("Wrong — correct answer: %s\n", q.Choices[q.A])
			}
			break
		}
		fmt.Println()
	}

	fmt.Printf("Done — your score: %d/%d\n", score, len(qs))
}
