# task-2
CREATE DATABASE survey_db;

USE survey_db;

CREATE TABLE survey_questions (
    question_id INT AUTO_INCREMENT PRIMARY KEY,
    question_text VARCHAR(255) NOT NULL
);

CREATE TABLE question_options (
    option_id INT AUTO_INCREMENT PRIMARY KEY,
    question_id INT NOT NULL,
    option_text VARCHAR(100) NOT NULL,
    FOREIGN KEY (question_id) REFERENCES survey_questions (question_id)
);

CREATE TABLE survey_responses (
    response_id INT AUTO_INCREMENT PRIMARY KEY,
    question_id INT NOT NULL,
    user_id INT NOT NULL, -- If you want to associate responses with users
    response_option_id INT NOT NULL,
    FOREIGN KEY (question_id) REFERENCES survey_questions (question_id),
    FOREIGN KEY (response_option_id) REFERENCES question_options (option_id)
);
// java code
// SurveyQuestion.java
@Entity
public class SurveyQuestion {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String questionText;

    // Getters and Setters
}

// QuestionOption.java
@Entity
public class QuestionOption {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    private SurveyQuestion question;

    private String optionText;

    // Getters and Setters
}

// SurveyResponse.java
@Entity
public class SurveyResponse {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    private SurveyQuestion question;

    @ManyToOne
    private QuestionOption responseOption;

    private Long userId; // Optional: If you want to associate responses with users

    // Getters and Setters
}

// SurveyRepository.java
public interface SurveyRepository extends JpaRepository<SurveyQuestion, Long> {
}

// QuestionOptionRepository.java
public interface QuestionOptionRepository extends JpaRepository<QuestionOption, Long> {
}

// SurveyResponseRepository.java
public interface SurveyResponseRepository extends JpaRepository<SurveyResponse, Long> {
}

// SurveyService.java
@Service
public class SurveyService {
    @Autowired
    private SurveyRepository surveyRepository;

    @Autowired
    private QuestionOptionRepository questionOptionRepository;

    @Autowired
    private SurveyResponseRepository surveyResponseRepository;

    public List<SurveyQuestion> getAllQuestions() {
        return surveyRepository.findAll();
    }

    public void saveResponse(Long questionId, Long optionId, Long userId) {
        SurveyQuestion question = surveyRepository.findById(questionId).orElse(null);
        QuestionOption option = questionOptionRepository.findById(optionId).orElse(null);

        if (question != null && option != null) {
            SurveyResponse response = new SurveyResponse();
            response.setQuestion(question);
            response.setResponseOption(option);
            response.setUserId(userId); // Optional: If you want to associate responses with users

            surveyResponseRepository.save(response);
        }
    }
}

// SurveyController.java
@RestController
public class SurveyController {
    @Autowired
    private SurveyService surveyService;

    @GetMapping("/survey/questions")
    public List<SurveyQuestion> getSurveyQuestions() {
        return surveyService.getAllQuestions();
    }

    @PostMapping("/survey/response")
    public void saveSurveyResponse(@RequestParam Long questionId, @RequestParam Long optionId, @RequestParam Long userId) {
        surveyService.saveResponse(questionId, optionId, userId);
    }
}

// Application.java (Main class)
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

