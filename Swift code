import SwiftUI
import Combine

// MARK: - Models

// Article Model for News
struct Article: Codable, Identifiable, Equatable {
    let id: UUID
    let title: String
    let description: String?
    let url: String
    let urlToImage: String?
    let publishedAt: String
    
    // Add computed property to format date nicely
    var formattedDate: String {
        let inputFormatter = DateFormatter()
        inputFormatter.dateFormat = "yyyy-MM-dd'T'HH:mm:ss'Z'"
        
        let outputFormatter = DateFormatter()
        outputFormatter.dateStyle = .medium
        
        if let date = inputFormatter.date(from: publishedAt) {
            return outputFormatter.string(from: date)
        }
        return publishedAt
    }
    
    // Implement Equatable
    static func == (lhs: Article, rhs: Article) -> Bool {
        return lhs.id == rhs.id
    }
    
    // Add initializer to support creating articles with a fixed ID
    init(id: UUID = UUID(), title: String, description: String?, url: String, urlToImage: String?, publishedAt: String) {
        self.id = id
        self.title = title
        self.description = description
        self.url = url
        self.urlToImage = urlToImage
        self.publishedAt = publishedAt
    }
}

// Certification Model for Displaying Certification Cards
struct Certification: Identifiable, Equatable {
    let id: UUID
    let imageName: String
    let redirectURL: URL
    let title: String
    let description: String
    
    init(id: UUID = UUID(), imageName: String, redirectURL: URL, title: String, description: String) {
        self.id = id
        self.imageName = imageName
        self.redirectURL = redirectURL
        self.title = title
        self.description = description
    }
}

// Question Model for the Quiz/Question Bank
struct Question: Identifiable, Codable, Equatable {
    let id: UUID
    let level: String
    let topic: String
    let questionText: String
    let options: [String]
    let correctAnswer: String
    let explanation: String
    
    // Implement Equatable
    static func == (lhs: Question, rhs: Question) -> Bool {
        return lhs.id == rhs.id
    }
    
    init(id: UUID = UUID(), level: String, topic: String, questionText: String, options: [String], correctAnswer: String, explanation: String = "") {
        self.id = id
        self.level = level
        self.topic = topic
        self.questionText = questionText
        self.options = options
        self.correctAnswer = correctAnswer
        self.explanation = explanation
    }
}

// User Progress Model
struct UserProgress: Codable {
    var completedQuizzes: [String: Int] // [quizId: score]
    var lastAccessedDate: Date
    
    init(completedQuizzes: [String: Int] = [:], lastAccessedDate: Date = Date()) {
        self.completedQuizzes = completedQuizzes
        self.lastAccessedDate = lastAccessedDate
    }
}

// MARK: - Services

// Networking Manager for News API
class NetworkManager: ObservableObject {
    @Published var articles: [Article] = []
    @Published var isLoading = false
    @Published var errorMessage: String?
    
    private var cancellables = Set<AnyCancellable>()
    
    // API Keys should be stored securely and fetched from a configuration file or environment
    private let apiKey = "<API_KEY>"
    
    func fetchJavaNews() {
        isLoading = true
        errorMessage = nil
        
        let urlString = "https://newsapi.org/v2/everything?q=java%20programming&language=en&sortBy=publishedAt&apiKey=\(apiKey)"
        guard let url = URL(string: urlString) else {
            self.errorMessage = "Invalid URL"
            self.isLoading = false
            return
        }
        
        URLSession.shared.dataTaskPublisher(for: url)
            .map { $0.data }
            .decode(type: NewsResponse.self, decoder: JSONDecoder())
            .receive(on: DispatchQueue.main)
            .sink { [weak self] completion in
                self?.isLoading = false
                if case let .failure(error) = completion {
                    self?.errorMessage = "Error: \(error.localizedDescription)"
                    print("Error fetching news: \(error)")
                }
            } receiveValue: { [weak self] response in
                self?.articles = response.articles
            }
            .store(in: &cancellables)
    }
}

// Helper for decoding the JSON response from NewsAPI
struct NewsResponse: Codable {
    let articles: [Article]
}

// Question Service to manage quiz data
class QuestionService: ObservableObject {
    @Published var questions: [Question] = []
    @Published var isLoading = false
    @Published var errorMessage: String?
    
    // In a real app, fetch from server or local JSON file
    func loadQuestions() {
        isLoading = true
        
        // Simulate network delay
        DispatchQueue.main.asyncAfter(deadline: .now() + 0.5) {
            self.questions = self.getMockQuestions()
            self.isLoading = false
        }
    }
    
    // Get questions filtered by level and topic
    func getQuestions(level: String, topic: String) -> [Question] {
        return questions.filter { $0.level == level && $0.topic == topic }
    }
    
    // Mock data - would normally be fetched from a server
    private func getMockQuestions() -> [Question] {
        return [
            Question(
                level: "Beginner",
                topic: "Syntax",
                questionText: "Which keyword is used to create an instance of a class in Java?",
                options: ["new", "make", "create", "instance"],
                correctAnswer: "new",
                explanation: "The 'new' keyword is used to create objects in Java by allocating memory for the object."
            ),
            Question(
                level: "Beginner",
                topic: "Syntax",
                questionText: "In Java, what is the correct way to declare a constant variable?",
                options: ["final int MAX_VALUE = 100;", "const int MAX_VALUE = 100;", "static int MAX_VALUE = 100;", "immutable int MAX_VALUE = 100;"],
                correctAnswer: "final int MAX_VALUE = 100;",
                explanation: "In Java, the 'final' keyword is used to declare constants. Once assigned, a final variable cannot be changed."
            ),
            Question(
                level: "Intermediate",
                topic: "OOP",
                questionText: "What is the main benefit of encapsulation in Java?",
                options: ["Code reusability", "Data hiding", "Multiple inheritance", "Method overloading"],
                correctAnswer: "Data hiding",
                explanation: "Encapsulation in Java is a mechanism of wrapping data (variables) and code acting on the data (methods) together as a single unit. It hides the internal state of objects from outside access."
            ),
            Question(
                level: "Advanced",
                topic: "Concurrency",
                questionText: "Which of the following is NOT a way to create a thread in Java?",
                options: ["Extending Thread class", "Implementing Runnable interface", "Implementing Callable interface", "Implementing Processable interface"],
                correctAnswer: "Implementing Processable interface",
                explanation: "Java provides several ways to create threads including extending Thread class, implementing Runnable or Callable interfaces, but there is no Processable interface in Java for thread creation."
            ),
            Question(
                level: "Intermediate",
                topic: "Collections",
                questionText: "Which collection type should be used when you need to ensure elements are unique?",
                options: ["ArrayList", "LinkedList", "HashSet", "PriorityQueue"],
                correctAnswer: "HashSet",
                explanation: "HashSet implements the Set interface which does not allow duplicate elements. It uses a hash table for storage."
            )
        ]
    }
}

// User Service to manage user data and progress
class UserService: ObservableObject {
    @Published var userProgress: UserProgress = UserProgress()
    
    private let userDefaultsKey = "userProgress"
    
    init() {
        loadUserProgress()
    }
    
    func saveQuizResult(quizId: String, score: Int) {
        userProgress.completedQuizzes[quizId] = score
        userProgress.lastAccessedDate = Date()
        saveUserProgress()
    }
    
    func getQuizScore(quizId: String) -> Int? {
        return userProgress.completedQuizzes[quizId]
    }
    
    private func loadUserProgress() {
        if let data = UserDefaults.standard.data(forKey: userDefaultsKey),
           let savedProgress = try? JSONDecoder().decode(UserProgress.self, from: data) {
            userProgress = savedProgress
        }
    }
    
    private func saveUserProgress() {
        if let encodedData = try? JSONEncoder().encode(userProgress) {
            UserDefaults.standard.set(encodedData, forKey: userDefaultsKey)
        }
    }
}

// CertificationService to manage certification data
class CertificationService: ObservableObject {
    @Published var certifications: [Certification] = []
    
    init() {
        loadCertifications()
    }
    
    private func loadCertifications() {
        // In a real app, fetch from server
        certifications = [
            Certification(
                imageName: "oracle_cert",
                redirectURL: URL(string: "https://education.oracle.com/java-certification")!,
                title: "Oracle Java Certification",
                description: "Validate your Java skills with globally recognized Oracle certification programs."
            ),
            Certification(
                imageName: "openjdk",
                redirectURL: URL(string: "https://openjdk.java.net/")!,
                title: "OpenJDK Certification",
                description: "Get certified in OpenJDK, the open-source implementation of the Java Platform."
            ),
            Certification(
                imageName: "spring",
                redirectURL: URL(string: "https://spring.io/training/certification")!,
                title: "Spring Framework Certification",
                description: "Become an expert in Java's popular Spring Framework with this certification."
            )
        ]
    }
}

// MARK: - View Models

// View Model for Quiz screens
class QuizViewModel: ObservableObject {
    @Published var currentQuestionIndex: Int = 0
    @Published var score: Int = 0
    @Published var selectedAnswer: String?
    @Published var showExplanation: Bool = false
    @Published var isQuizCompleted: Bool = false
    @Published var questions: [Question] = []
    
    private let userService: UserService
    
    init(userService: UserService) {
        self.userService = userService
    }
    
    var currentQuestion: Question? {
        guard !questions.isEmpty, currentQuestionIndex < questions.count else { return nil }
        return questions[currentQuestionIndex]
    }
    
    var progressPercentage: Double {
        guard !questions.isEmpty else { return 0 }
        return Double(currentQuestionIndex) / Double(questions.count)
    }
    
    var quizId: String {
        if let firstQuestion = questions.first {
            return "\(firstQuestion.level)-\(firstQuestion.topic)"
        }
        return "unknown-quiz"
    }
    
    func startQuiz(with questions: [Question]) {
        self.questions = questions
        currentQuestionIndex = 0
        score = 0
        isQuizCompleted = false
        selectedAnswer = nil
        showExplanation = false
    }
    
    func selectAnswer(_ answer: String) {
        guard !showExplanation else { return }
        
        selectedAnswer = answer
        
        if answer == currentQuestion?.correctAnswer {
            score += 1
        }
        
        showExplanation = true
    }
    
    func nextQuestion() {
        if currentQuestionIndex < questions.count - 1 {
            currentQuestionIndex += 1
            selectedAnswer = nil
            showExplanation = false
        } else {
            completeQuiz()
        }
    }
    
    private func completeQuiz() {
        isQuizCompleted = true
        userService.saveQuizResult(quizId: quizId, score: score)
    }
    
    func retakeQuiz() {
        startQuiz(with: questions)
    }
}

// View Model for Coding Practice
class CodingPracticeViewModel: ObservableObject {
    @Published var codeInput: String = "// Write your Java code here\npublic class HelloWorld {\n    public static void main(String[] args) {\n        System.out.println(\"Hello, Java!\");\n    }\n}"
    @Published var output: String = "Output will be displayed here..."
    @Published var isRunning: Bool = false
    @Published var savedSnippets: [String] = []
    
    private let userDefaultsKey = "savedCodeSnippets"
    
    init() {
        loadSavedSnippets()
    }
    
    func runCode() {
        isRunning = true
        
        // In a real app, send to a backend for compilation and execution
        // Here we simulate a delay and response
        DispatchQueue.main.asyncAfter(deadline: .now() + 1.5) {
            self.simulateCodeExecution()
            self.isRunning = false
        }
    }
    
    func saveSnippet() {
        if !codeInput.isEmpty && !savedSnippets.contains(codeInput) {
            savedSnippets.append(codeInput)
            UserDefaults.standard.set(savedSnippets, forKey: userDefaultsKey)
        }
    }
    
    func loadSnippet(_ snippet: String) {
        codeInput = snippet
    }
    
    private func loadSavedSnippets() {
        if let snippets = UserDefaults.standard.stringArray(forKey: userDefaultsKey) {
            savedSnippets = snippets
        }
    }
    
    private func simulateCodeExecution() {
        if codeInput.contains("System.out.println") {
            // Extract the content inside println
            if let range = codeInput.range(of: "System.out.println\\(\\s*\"(.+?)\"\\s*\\)", options: .regularExpression) {
                let match = String(codeInput[range])
                if let contentRange = match.range(of: "\"(.+?)\"", options: .regularExpression) {
                    let content = String(match[contentRange])
                    output = "Program output:\n" + content.replacingOccurrences(of: "\"", with: "")
                    return
                }
            }
        }
        
        // If no println found or couldn't extract content
        if codeInput.contains("public static void main") {
            output = "Program executed successfully with no visible output."
        } else {
            output = "Error: Missing main method. Java programs must contain a main method."
        }
    }
}

// MARK: - SwiftUI Views

// Home View (News + Certification Cards)
struct HomeView: View {
    @StateObject var networkManager = NetworkManager()
    @StateObject var certificationService = CertificationService()
    @ObservedObject var userService: UserService
    
    var body: some View {
        NavigationView {
            ScrollView {
                VStack(alignment: .leading, spacing: 20) {
                    
                    // Welcome section
                    VStack(alignment: .leading) {
                        Text("Welcome to Java Quiz & News")
                            .font(.largeTitle)
                            .fontWeight(.bold)
                            .padding(.horizontal)
                        
                        Text("Learn Java through interactive quizzes, stay updated with the latest news, and practice coding.")
                            .padding(.horizontal)
                            .foregroundColor(.secondary)
                    }
                    .padding(.top)
                    
                    // MARK: News Section
                    SectionHeader(title: "Latest Java News", systemImage: "newspaper.fill")
                    
                    if networkManager.isLoading {
                        ProgressView("Loading news...")
                            .frame(maxWidth: .infinity, alignment: .center)
                            .padding()
                    } else if let errorMessage = networkManager.errorMessage {
                        ErrorView(message: errorMessage, retryAction: {
                            networkManager.fetchJavaNews()
                        })
                    } else if networkManager.articles.isEmpty {
                        Text("No articles available")
                            .frame(maxWidth: .infinity, alignment: .center)
                            .padding()
                    } else {
                        ScrollView(.horizontal, showsIndicators: false) {
                            HStack(spacing: 16) {
                                ForEach(networkManager.articles) { article in
                                    NavigationLink(destination: ArticleDetailView(article: article)) {
                                        NewsCardView(article: article)
                                            .frame(width: 300, height: 220)
                                    }
                                }
                            }
                            .padding(.horizontal)
                        }
                    }
                    
                    // MARK: Certifications Section
                    SectionHeader(title: "Global Certifications", systemImage: "certificate.fill")
                    
                    ScrollView(.horizontal, showsIndicators: false) {
                        HStack(spacing: 16) {
                            ForEach(certificationService.certifications) { cert in
                                CertificationCardView(certification: cert)
                            }
                        }
                        .padding(.horizontal)
                    }
                    
                    // MARK: Your Progress Section
                    SectionHeader(title: "Your Progress", systemImage: "chart.bar.fill")
                    
                    VStack(alignment: .leading) {
                        if userService.userProgress.completedQuizzes.isEmpty {
                            Text("You haven't completed any quizzes yet.")
                                .padding(.horizontal)
                                .foregroundColor(.secondary)
                        } else {
                            ProgressSummaryView(userService: userService)
                                .padding(.horizontal)
                        }
                        
                        NavigationLink(destination: QuestionBankView(userService: userService)) {
                            HStack {
                                Text("Take a quiz")
                                Spacer()
                                Image(systemName: "chevron.right")
                            }
                            .padding()
                            .background(Color.blue)
                            .foregroundColor(.white)
                            .cornerRadius(10)
                            .padding(.horizontal)
                        }
                    }
                }
                .padding(.bottom, 20)
            }
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .principal) {
                    Image(systemName: "graduationcap.fill")
                        .foregroundColor(.blue)
                }
            }
            .refreshable {
                networkManager.fetchJavaNews()
            }
            .onAppear {
                networkManager.fetchJavaNews()
            }
        }
    }
}

// Section Header Component
struct SectionHeader: View {
    let title: String
    let systemImage: String
    
    var body: some View {
        HStack {
            Image(systemName: systemImage)
                .foregroundColor(.blue)
            Text(title)
                .font(.title2)
                .fontWeight(.bold)
        }
        .padding(.horizontal)
        .padding(.top, 10)
    }
}

// Progress Summary Component
struct ProgressSummaryView: View {
    @ObservedObject var userService: UserService
    
    var body: some View {
        VStack(alignment: .leading, spacing: 10) {
            Text("Completed Quizzes: \(userService.userProgress.completedQuizzes.count)")
                .font(.headline)
            
            if let averageScore = calculateAverageScore() {
                Text("Average Score: \(averageScore, specifier: "%.1f")%")
                    .font(.subheadline)
            }
            
            Text("Last studied: \(userService.userProgress.lastAccessedDate, style: .date)")
                .font(.caption)
                .foregroundColor(.secondary)
        }
        .padding()
        .background(Color.blue.opacity(0.1))
        .cornerRadius(10)
    }
    
    private func calculateAverageScore() -> Double? {
        let scores = userService.userProgress.completedQuizzes.values
        guard !scores.isEmpty else { return nil }
        let sum = Double(scores.reduce(0, +))
        return (sum / Double(scores.count)) * 100
    }
}

// Error View Component
struct ErrorView: View {
    let message: String
    let retryAction: () -> Void
    
    var body: some View {
        VStack(spacing: 15) {
            Image(systemName: "exclamationmark.triangle")
                .font(.largeTitle)
                .foregroundColor(.orange)
            
            Text(message)
                .multilineTextAlignment(.center)
            
            Button("Retry") {
                retryAction()
            }
            .padding(.horizontal, 20)
            .padding(.vertical, 10)
            .background(Color.blue)
            .foregroundColor(.white)
            .cornerRadius(8)
        }
        .padding()
        .frame(maxWidth: .infinity)
        .background(Color.gray.opacity(0.1))
        .cornerRadius(12)
        .padding()
    }
}

// Enhanced News Card View
struct NewsCardView: View {
    let article: Article
    
    var body: some View {
        VStack(alignment: .leading) {
            if let imageUrlString = article.urlToImage, let imageUrl = URL(string: imageUrlString) {
                AsyncImage(url: imageUrl) { phase in
                    switch phase {
                    case .empty:
                        Rectangle()
                            .foregroundColor(Color.gray.opacity(0.3))
                            .shimmer()
                    case .success(let image):
                        image
                            .resizable()
                            .aspectRatio(contentMode: .fill)
                    case .failure:
                        Image(systemName: "photo")
                            .foregroundColor(Color.gray)
                            .frame(maxWidth: .infinity, alignment: .center)
                    @unknown default:
                        EmptyView()
                    }
                }
                .frame(height: 140)
                .clipped()
                .cornerRadius(8)
            } else {
                Rectangle()
                    .foregroundColor(Color.gray.opacity(0.3))
                    .frame(height: 140)
                    .cornerRadius(8)
                    .overlay(
                        Image(systemName: "newspaper")
                            .foregroundColor(.gray)
                    )
            }
            
            VStack(alignment: .leading, spacing: 4) {
                Text(article.title)
                    .font(.headline)
                    .lineLimit(2)
                    .foregroundColor(.primary)
                
                Text(article.formattedDate)
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
            .padding(.horizontal, 8)
            .padding(.vertical, 6)
        }
        .background(Color(UIColor.systemBackground))
        .cornerRadius(10)
        .shadow(color: Color.black.opacity(0.2), radius: 5)
    }
}

// Article Detail View with more functionality
struct ArticleDetailView: View {
    let article: Article
    @State private var showShareSheet = false
    
    var body: some View {
        ScrollView {
            VStack(alignment: .leading, spacing: 16) {
                if let imageUrlString = article.urlToImage, let url = URL(string: imageUrlString) {
                    AsyncImage(url: url) { phase in
                        switch phase {
                        case .empty:
                            Rectangle()
                                .foregroundColor(Color.gray.opacity(0.3))
                                .shimmer()
                        case .success(let image):
                            image
                                .resizable()
                                .aspectRatio(contentMode: .fill)
                        case .failure:
                            Rectangle()
                                .foregroundColor(Color.gray.opacity(0.3))
                                .overlay(
                                    Image(systemName: "photo")
                                        .foregroundColor(.gray)
                                )
                        @unknown default:
                            EmptyView()
                        }
                    }
                    .aspectRatio(16/9, contentMode: .fit)
                    .cornerRadius(8)
                }
                
                VStack(alignment: .leading, spacing: 12) {
                    Text(article.title)
                        .font(.title)
                        .fontWeight(.bold)
                    
                    Text(article.formattedDate)
                        .font(.subheadline)
                        .foregroundColor(.secondary)
                    
                    Divider()
                    
                    Text(article.description ?? "No description available.")
                        .font(.body)
                    
                    if let url = URL(string: article.url) {
                        Link(destination: url) {
                            HStack {
                                Text("Read full article")
                                Image(systemName: "arrow.up.right.square")
                            }
                            .font(.headline)
                            .foregroundColor(.blue)
                            .padding(.top, 8)
                        }
                    }
                }
                .padding()
            }
        }
        .navigationTitle("Article")
        .navigationBarTitleDisplayMode(.inline)
        .toolbar {
            ToolbarItem(placement: .navigationBarTrailing) {
                Button(action: {
                    showShareSheet = true
                }) {
                    Image(systemName: "square.and.arrow.up")
                }
            }
        }
        .sheet(isPresented: $showShareSheet) {
            if let url = URL(string: article.url) {
                ShareSheet(items: [url])
            }
        }
    }
}

// ShareSheet for sharing content
struct ShareSheet: UIViewControllerRepresentable {
    let items: [Any]
    
    func makeUIViewController(context: Context) -> UIActivityViewController {
        let controller = UIActivityViewController(activityItems: items, applicationActivities: nil)
        return controller
    }
    
    func updateUIViewController(_ uiViewController: UIActivityViewController, context: Context) {}
}

// Enhanced Certification Card View
struct CertificationCardView: View {
    let certification: Certification
    
    var body: some View {
        VStack(alignment: .leading) {
            // Using a local image or placeholder
            ZStack {
                Rectangle()
                    .foregroundColor(Color.blue.opacity(0.2))
                    .frame(height: 100)
                    .cornerRadius(8)
                
                Image(certification.imageName)
                    .resizable()
                    .scaledToFit()
                    .frame(height: 70)
                    .cornerRadius(6)
            }
            
            VStack(alignment: .leading, spacing: 4) {
                Text(certification.title)
                    .font(.headline)
                    .lineLimit(1)
                
                Text(certification.description)
                    .font(.caption)
                    .foregroundColor(.secondary)
                    .lineLimit(2)
                    .frame(width: 200)
                
                Button(action: {
                    UIApplication.shared.open(certification.redirectURL)
                }) {
                    Text("Learn More")
                        .font(.caption)
                        .padding(.horizontal, 10)
                        .padding(.vertical, 5)
                        .background(Color.blue)
                        .foregroundColor(.white)
                        .cornerRadius(12)
                }
                .padding(.top, 5)
            }
            .padding(8)
        }
        .frame(width: 220)
        .background(Color(UIColor.systemBackground))
        .cornerRadius(12)
        .shadow(color: Color.black.opacity(0.1), radius: 5)
    }
}

// Enhanced Question Bank View
struct QuestionBankView: View {
    @StateObject var questionService = QuestionService()
    @ObservedObject var userService: UserService
    
    let topics = ["Syntax", "OOP", "Collections", "Concurrency"]
    let levels = ["Beginner", "Intermediate", "Advanced"]
    
    var body: some View {
        ScrollView {
            VStack(alignment: .leading, spacing: 20) {
                SectionHeader(title: "Java Question Bank", systemImage: "list.bullet.clipboard")
                    .padding(.top)
                
                if questionService.isLoading {
                    ProgressView("Loading questions...")
                        .frame(maxWidth: .infinity, alignment: .center)
                        .padding()
                } else if let errorMessage = questionService.errorMessage {
                    ErrorView(message: errorMessage) {
                        questionService.loadQuestions()
                    }
                } else {
                    ForEach(levels, id: \.self) { level in
                        VStack(alignment: .leading) {
                            Text(level)
                                .font(.headline)
                                .padding(.horizontal)
                            
                            ForEach(topics, id: \.self) { topic in
                                let quizId = "\(level)-\(topic)"
                                let questions = questionService.getQuestions(level: level, topic: topic)
                                let hasTaken = userService.getQuizScore(quizId: quizId) != nil
                                
                                if !questions.isEmpty {
                                    NavigationLink(destination: QuizView(
                                        level: level,
                                        topic: topic,
                                        questionService: questionService,
                                        userService: userService
                                    )) {
                                        QuizItemRow(
                                            topic: topic,
                                            count: questions.count,
                                            hasTaken: hasTaken,
                                            score: userService.getQuizScore(quizId: quizId)
                                        )
                                        .padding(.vertical, 8)
                                        .padding(.horizontal)
                                        .background(Color(UIColor.systemBackground))
                                        .cornerRadius(10)
                                        .shadow(color: Color.black.opacity(0.05), radius: 2)
                                        .padding(.horizontal)
                                    }
                                }
                            }
                        }
                        .padding(.bottom, 10)
                    }
                }
            }
            .padding(.bottom, 20)
        }
        .navigationTitle("Question Bank")
        .onAppear {
            questionService.loadQuestions()
        }
    }
}

// Quiz Item Row Component
struct QuizItemRow: View {
    let topic: String
    let count: Int
    let hasTaken: Bool
    let score: Int?
    
    var body: some View {
        HStack {
            VStack(alignment: .leading, spacing: 4) {
                Text(topic)
                    .font(.headline)
                
                Text("\(count) questions")
                    .font(.subheadline)
                    .foregroundColor(.secondary)
            }
            
            Spacer()
            
            if hasTaken, let score = score {
                Text("\(score)/\(count)")
                    .fontWeight(.bold)
                    .foregroundColor(.blue)
            } else {
                Text("Start")
                    .fontWeight(.semibold)
                    .padding(.horizontal, 15)
                    .padding(.vertical, 5)
                    .background(Color.blue)
                    .foregroundColor(.white)
                                        .cornerRadius(8)
                                }
                                
                                Image(systemName: "chevron.right")
                                    .foregroundColor(.blue)
                                    .padding(.leading, 5)
                            }
                        }
                    }

                    // Enhanced Quiz View
                    struct QuizView: View {
                        let level: String
                        let topic: String
                        @ObservedObject var questionService: QuestionService
                        @ObservedObject var userService: UserService
                        @StateObject private var viewModel: QuizViewModel
                        
                        init(level: String, topic: String, questionService: QuestionService, userService: UserService) {
                            self.level = level
                            self.topic = topic
                            self.questionService = questionService
                            self.userService = userService
                            _viewModel = StateObject(wrappedValue: QuizViewModel(userService: userService))
                        }
                        
                        var body: some View {
                            ZStack {
                                // Background
                                Color(UIColor.systemGroupedBackground)
                                    .ignoresSafeArea()
                                
                                if viewModel.isQuizCompleted {
                                    QuizCompletionView(
                                        score: viewModel.score,
                                        totalQuestions: viewModel.questions.count,
                                        retakeAction: {
                                            viewModel.retakeQuiz()
                                        },
                                        browseAction: {
                                            // This will be handled by NavigationView popping back
                                        }
                                    )
                                } else if let question = viewModel.currentQuestion {
                                    VStack(spacing: 0) {
                                        // Progress bar
                                        ProgressView(value: viewModel.progressPercentage)
                                            .padding()
                                        
                                        ScrollView {
                                            VStack(alignment: .leading, spacing: 20) {
                                                // Question header
                                                Text("Question \(viewModel.currentQuestionIndex + 1) of \(viewModel.questions.count)")
                                                    .font(.headline)
                                                    .foregroundColor(.secondary)
                                                    .padding(.horizontal)
                                                    .padding(.top)
                                                
                                                // Question text
                                                Text(question.questionText)
                                                    .font(.title3)
                                                    .fontWeight(.semibold)
                                                    .padding(.horizontal)
                                                
                                                // Answer options
                                                ForEach(question.options, id: \.self) { option in
                                                    Button {
                                                        viewModel.selectAnswer(option)
                                                    } label: {
                                                        HStack {
                                                            Text(option)
                                                                .padding()
                                                                .frame(maxWidth: .infinity, alignment: .leading)
                                                                .multilineTextAlignment(.leading)
                                                            
                                                            if viewModel.showExplanation {
                                                                if option == question.correctAnswer {
                                                                    Image(systemName: "checkmark.circle.fill")
                                                                        .foregroundColor(.green)
                                                                } else if option == viewModel.selectedAnswer {
                                                                    Image(systemName: "xmark.circle.fill")
                                                                        .foregroundColor(.red)
                                                                }
                                                            }
                                                        }
                                                        .background(
                                                            RoundedRectangle(cornerRadius: 10)
                                                                .fill(optionBackgroundColor(option))
                                                        )
                                                    }
                                                    .disabled(viewModel.showExplanation)
                                                    .foregroundColor(optionTextColor(option))
                                                    .padding(.horizontal)
                                                }
                                                
                                                // Explanation section
                                                if viewModel.showExplanation {
                                                    VStack(alignment: .leading, spacing: 10) {
                                                        Text("Explanation")
                                                            .font(.headline)
                                                        
                                                        Text(question.explanation)
                                                            .font(.body)
                                                        
                                                        Button {
                                                            viewModel.nextQuestion()
                                                        } label: {
                                                            Text(viewModel.currentQuestionIndex < viewModel.questions.count - 1 ? "Next Question" : "Finish Quiz")
                                                                .fontWeight(.bold)
                                                                .padding()
                                                                .frame(maxWidth: .infinity)
                                                                .background(Color.blue)
                                                                .foregroundColor(.white)
                                                                .cornerRadius(10)
                                                        }
                                                        .padding(.top)
                                                    }
                                                    .padding()
                                                    .background(Color(UIColor.secondarySystemBackground))
                                                    .cornerRadius(10)
                                                    .padding(.horizontal)
                                                    .padding(.top, 10)
                                                    .transition(.move(edge: .bottom).combined(with: .opacity))
                                                }
                                                
                                                Spacer(minLength: 40)
                                            }
                                            .padding(.bottom, 20)
                                        }
                                    }
                                    .animation(.easeInOut, value: viewModel.showExplanation)
                                } else {
                                    // No questions available
                                    VStack(spacing: 20) {
                                        Image(systemName: "exclamationmark.triangle")
                                            .font(.largeTitle)
                                            .foregroundColor(.orange)
                                        
                                        Text("No questions available for this topic.")
                                            .font(.headline)
                                        
                                        Button("Go Back") {
                                            // This will be handled by NavigationView
                                        }
                                        .padding()
                                        .background(Color.blue)
                                        .foregroundColor(.white)
                                        .cornerRadius(10)
                                    }
                                }
                            }
                            .navigationTitle("\(topic) Quiz - \(level)")
                            .onAppear {
                                let questions = questionService.getQuestions(level: level, topic: topic)
                                viewModel.startQuiz(with: questions)
                            }
                        }
                        
                        private func optionBackgroundColor(_ option: String) -> Color {
                            guard viewModel.showExplanation else {
                                return viewModel.selectedAnswer == option ? Color.blue.opacity(0.2) : Color(UIColor.secondarySystemBackground)
                            }
                            
                            if option == viewModel.currentQuestion?.correctAnswer {
                                return Color.green.opacity(0.2)
                            } else if option == viewModel.selectedAnswer {
                                return Color.red.opacity(0.2)
                            }
                            return Color(UIColor.secondarySystemBackground)
                        }
                        
                        private func optionTextColor(_ option: String) -> Color {
                            if !viewModel.showExplanation {
                                return .primary
                            }
                            
                            if option == viewModel.currentQuestion?.correctAnswer {
                                return .green
                            } else if option == viewModel.selectedAnswer && option != viewModel.currentQuestion?.correctAnswer {
                                return .red
                            }
                            return .primary
                        }
                    }

                    // Quiz Completion View
                    struct QuizCompletionView: View {
                        let score: Int
                        let totalQuestions: Int
                        let retakeAction: () -> Void
                        let browseAction: () -> Void
                        
                        var body: some View {
                            VStack(spacing: 30) {
                                Image(systemName: "checkmark.circle.fill")
                                    .font(.system(size: 80))
                                    .foregroundColor(.green)
                                
                                Text("Quiz Completed!")
                                    .font(.largeTitle)
                                    .fontWeight(.bold)
                                
                                VStack(spacing: 15) {
                                    Text("Your Score")
                                        .font(.headline)
                                        .foregroundColor(.secondary)
                                    
                                    Text("\(score)/\(totalQuestions)")
                                        .font(.system(size: 44, weight: .bold))
                                    
                                    Text("(\(Int((Double(score) / Double(totalQuestions)) * 100))%)")
                                        .font(.title)
                                        .foregroundColor(.secondary)
                                }
                                
                                // Performance feedback
                                Text(performanceFeedback())
                                    .font(.headline)
                                    .foregroundColor(performanceColor())
                                    .padding()
                                    .frame(maxWidth: .infinity)
                                    .background(performanceColor().opacity(0.1))
                                    .cornerRadius(10)
                                    .padding(.horizontal)
                                
                                VStack(spacing: 15) {
                                    Button {
                                        retakeAction()
                                    } label: {
                                        HStack {
                                            Image(systemName: "arrow.clockwise")
                                            Text("Retake Quiz")
                                        }
                                        .padding()
                                        .frame(maxWidth: .infinity)
                                        .background(Color.blue)
                                        .foregroundColor(.white)
                                        .cornerRadius(10)
                                    }
                                    
                                    Button {
                                        browseAction()
                                    } label: {
                                        HStack {
                                            Image(systemName: "list.bullet")
                                            Text("Browse More Quizzes")
                                        }
                                        .padding()
                                        .frame(maxWidth: .infinity)
                                        .background(Color.blue.opacity(0.1))
                                        .foregroundColor(.blue)
                                        .cornerRadius(10)
                                    }
                                }
                                .padding(.horizontal)
                            }
                            .padding()
                        }
                        
                        private func performanceFeedback() -> String {
                            let percentage = Double(score) / Double(totalQuestions)
                            if percentage >= 0.9 {
                                return "Excellent! You've mastered this topic."
                            } else if percentage >= 0.7 {
                                return "Good job! You're doing well."
                            } else if percentage >= 0.5 {
                                return "Not bad, but there's room for improvement."
                            } else {
                                return "Keep practicing! You'll get better."
                            }
                        }
                        
                        private func performanceColor() -> Color {
                            let percentage = Double(score) / Double(totalQuestions)
                            if percentage >= 0.9 {
                                return .green
                            } else if percentage >= 0.7 {
                                return .blue
                            } else if percentage >= 0.5 {
                                return .orange
                            } else {
                                return .red
                            }
                        }
                    }

                    // Enhanced Coding Practice View
                    struct CodingPracticeView: View {
                        @StateObject private var viewModel = CodingPracticeViewModel()
                        @State private var showSavedSnippets = false
                        
                        var body: some View {
                            NavigationView {
                                VStack(spacing: 0) {
                                    CodeEditorView(text: $viewModel.codeInput)
                                        .frame(height: UIScreen.main.bounds.height * 0.4)
                                    
                                    Divider()
                                    
                                    VStack(alignment: .leading, spacing: 0) {
                                        HStack {
                                            Text("Output")
                                                .font(.headline)
                                                .padding(.horizontal)
                                            
                                            Spacer()
                                            
                                            Button(action: {
                                                viewModel.runCode()
                                            }) {
                                                HStack {
                                                    Image(systemName: "play.fill")
                                                    Text("Run")
                                                }
                                                .padding(.horizontal, 16)
                                                .padding(.vertical, 8)
                                                .background(Color.green)
                                                .foregroundColor(.white)
                                                .cornerRadius(8)
                                            }
                                            .disabled(viewModel.isRunning)
                                            .padding(.horizontal)
                                        }
                                        .padding(.vertical, 8)
                                        
                                        // Output panel
                                        OutputPanel(output: viewModel.output, isRunning: viewModel.isRunning)
                                            .frame(maxHeight: .infinity)
                                    }
                                }
                                .navigationTitle("Java Coding Practice")
                                .navigationBarTitleDisplayMode(.inline)
                                .toolbar {
                                    ToolbarItem(placement: .navigationBarLeading) {
                                        Button(action: {
                                            showSavedSnippets = true
                                        }) {
                                            Image(systemName: "folder")
                                        }
                                    }
                                    
                                    ToolbarItem(placement: .navigationBarTrailing) {
                                        Button(action: {
                                            viewModel.saveSnippet()
                                        }) {
                                            Image(systemName: "square.and.arrow.down")
                                        }
                                    }
                                }
                                .sheet(isPresented: $showSavedSnippets) {
                                    SavedSnippetsView(snippets: viewModel.savedSnippets) { snippet in
                                        viewModel.loadSnippet(snippet)
                                        showSavedSnippets = false
                                    }
                                }
                            }
                        }
                    }

                    // Output Panel for code execution results
                    struct OutputPanel: View {
                        let output: String
                        let isRunning: Bool
                        
                        var body: some View {
                            ZStack {
                                ScrollView {
                                    Text(output)
                                        .font(.system(.body, design: .monospaced))
                                        .padding()
                                        .frame(maxWidth: .infinity, alignment: .leading)
                                }
                                .background(Color(UIColor.systemBackground))
                                
                                if isRunning {
                                    ProgressView("Running code...")
                                        .padding()
                                        .background(Color(UIColor.systemBackground))
                                        .cornerRadius(10)
                                        .shadow(radius: 5)
                                }
                            }
                        }
                    }

                    // Code Editor View
                    struct CodeEditorView: View {
                        @Binding var text: String
                        
                        var body: some View {
                            VStack(alignment: .leading, spacing: 0) {
                                HStack {
                                    Text("Java Code")
                                        .font(.headline)
                                    Spacer()
                                }
                                .padding(.horizontal)
                                .padding(.vertical, 8)
                                .background(Color(UIColor.systemGroupedBackground))
                                
                                TextEditor(text: $text)
                                    .font(.system(.body, design: .monospaced))
                                    .padding(4)
                                    .background(Color(UIColor.systemBackground))
                            }
                        }
                    }

                    // Saved Snippets View
                    struct SavedSnippetsView: View {
                        let snippets: [String]
                        let onSelect: (String) -> Void
                        @Environment(\.presentationMode) var presentationMode
                        
                        var body: some View {
                            NavigationView {
                                List {
                                    if snippets.isEmpty {
                                        Text("No saved snippets")
                                            .foregroundColor(.secondary)
                                            .frame(maxWidth: .infinity, alignment: .center)
                                            .padding()
                                    } else {
                                        ForEach(snippets.indices, id: \.self) { index in
                                            Button {
                                                onSelect(snippets[index])
                                            } label: {
                                                VStack(alignment: .leading) {
                                                    Text("Snippet \(index + 1)")
                                                        .font(.headline)
                                                    Text(snippetPreview(snippets[index]))
                                                        .font(.system(.caption, design: .monospaced))
                                                        .foregroundColor(.secondary)
                                                        .lineLimit(2)
                                                }
                                                .padding(.vertical, 4)
                                            }
                                        }
                                    }
                                }
                                .navigationTitle("Saved Snippets")
                                .navigationBarTitleDisplayMode(.inline)
                                .toolbar {
                                    ToolbarItem(placement: .navigationBarTrailing) {
                                        Button("Close") {
                                            presentationMode.wrappedValue.dismiss()
                                        }
                                    }
                                }
                            }
                        }
                        
                        private func snippetPreview(_ snippet: String) -> String {
                            let lines = snippet.components(separatedBy: .newlines)
                            let firstLine = lines.first(where: { !$0.trimmingCharacters(in: .whitespacesAndNewlines).isEmpty }) ?? ""
                            return firstLine
                        }
                    }

                    // MARK: - Helper Extensions

                    // Shimmer effect for loading states
                    struct ShimmerEffect: ViewModifier {
                        @State private var phase: CGFloat = 0
                        
                        func body(content: Content) -> some View {
                            content
                                .overlay(
                                    GeometryReader { geometry in
                                        LinearGradient(
                                            gradient: Gradient(stops: [
                                                .init(color: .clear, location: phase - 0.2),
                                                .init(color: .white.opacity(0.5), location: phase),
                                                .init(color: .clear, location: phase + 0.2)
                                            ]),
                                            startPoint: .leading,
                                            endPoint: .trailing
                                        )
                                        .blendMode(.plusLighter)
                                        .mask(content)
                                        .animation(
                                            Animation.linear(duration: 1.5)
                                                .repeatForever(autoreverses: false),
                                            value: phase
                                        )
                                        .onAppear {
                                            phase = 1
                                        }
                                    }
                                )
                        }
                    }

                    extension View {
                        func shimmer() -> some View {
                            self.modifier(ShimmerEffect())
                        }
                    }

                    // MARK: - Main App Entry

                    @main
                    struct JavaQuizApp: App {
                        @StateObject private var userService = UserService()
                        
                        var body: some Scene {
                            WindowGroup {
                                TabView {
                                    HomeView(userService: userService)
                                        .tabItem {
                                            Label("Home", systemImage: "house.fill")
                                        }
                                    QuestionBankView(userService: userService)
                                        .tabItem {
                                            Label("Quizzes", systemImage: "questionmark.circle.fill")
                                        }
                                    CodingPracticeView()
                                        .tabItem {
                                            Label("Code", systemImage: "laptopcomputer")
                                        }
                                }
                                .accentColor(.blue)
                            }
                        }
                    }
