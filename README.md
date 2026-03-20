import React, { useState, useEffect, useMemo } from 'react';
import { 
  BookOpen,
  ArrowRight,
  ArrowLeft,
  Timer,
  ChevronLeft,
  CheckCircle2,
  XCircle,
  LayoutGrid,
  Send,
  Dna,
  Trophy,
  Target,
  Clock,
  Sparkles,
  Calendar,
  Info,
  User,
  Shield,
  Search,
  AlertTriangle
} from 'lucide-react';
import { initializeApp } from 'firebase/app';
import { 
  getFirestore, 
  collection, 
  addDoc, 
  getDocs, 
  query, 
  serverTimestamp,
  orderBy
} from 'firebase/firestore';
import { getAuth, signInAnonymously, onAuthStateChanged, signInWithCustomToken } from 'firebase/auth';

// --- FIREBASE INITIALIZATION ---
const firebaseConfig = JSON.parse(__firebase_config);
const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);
const appId = typeof __app_id !== 'undefined' ? __app_id : 'beacon-brains-bio-2026';

// --- FULL DATA SOURCE (40 QUESTIONS) ---
const RAW_QUESTIONS = [
  {
    text: "Which of the following blood vessels carries oxygenated blood away from the heart?",
    options: ["Venules", "Veins", "Arteries", "Capillaries"],
    correctIndex: 2,
    explanation: "Arteries are the blood vessels that carry oxygenated blood away from the heart to the body's tissues. The only exception is the pulmonary artery, which carries deoxygenated blood from the heart to the lungs."
  },
  {
    text: "Which of the following statements is true regarding sexual reproduction in organisms?",
    options: [
      "It is a form of asexual reproduction where offspring are produced without the involvement of gametes.",
      "It does not involve the formation of gametes or the fusion of reproductive cells.",
      "It involves the production of offspring through a single parent, resulting in genetically identical offspring.",
      "It involves the fusion of gametes from two parents, resulting in offspring with genetic variation."
    ],
    correctIndex: 3,
    explanation: "Sexual reproduction involves the fusion of gametes (reproductive cells) from two parents, resulting in offspring with genetic variation."
  },
  {
    text: "Which of the following statements best describes the role of competition in the process of adaptation?",
    options: [
      "Competition ensures equal distribution of resources among individuals in a population.",
      "Competition reduces the need for adaptations as individuals coexist peacefully.",
      "Competition leads to the development of new traits and adaptations in a population.",
      "Competition leads to the selection of individuals with favorable traits for survival and reproduction."
    ],
    correctIndex: 3,
    explanation: "Competition leads to the selection of individuals with favorable traits (natural selection). These individuals are better equipped to survive and pass on those traits."
  },
  {
    text: "Digestive enzymes are responsible for",
    options: ["Breaking down food into smaller molecules.", "Absorbing nutrients into the bloodstream.", "Regulating the pH of the digestive tract.", "Transporting food through the digestive system."],
    correctIndex: 0,
    explanation: "Digestive enzymes are specialized proteins that break down large, complex food molecules into smaller, simpler ones that can be absorbed."
  },
  {
    text: "Which of the following statements is true about the kingdom Fungi?",
    options: ["Fungi are photosynthetic organisms.", "Fungi obtain nutrients by absorbing organic matter.", "Fungi are multicellular organisms.", "Fungi reproduce through the formation of seeds."],
    correctIndex: 1,
    explanation: "Fungi are heterotrophic; they release enzymes to break down organic matter and then absorb the nutrients."
  },
  {
    text: "Ecological succession refers to",
    options: ["The movement of organisms from one habitat to another.", "The gradual and predictable change in a community over time.", "The competition among species for limited resources.", "The process of natural selection in a population."],
    correctIndex: 1,
    explanation: "Succession is the predictable change in species composition over time until a stable climax community is reached."
  },
  {
    text: "Which of the following is the most inclusive level of classification in the Linnaean system?",
    options: ["Kingdom", "Order", "Class", "Phylum"],
    correctIndex: 0,
    explanation: "Kingdom is the broadest (most inclusive) level among the options provided in the hierarchy: Kingdom, Phylum, Class, Order, Family, Genus, Species."
  },
  {
    text: "Which of the following best describes the concept of trophic levels in a functioning ecosystem?",
    options: ["The levels of nutrient cycling within an ecosystem.", "The levels of biological diversity within an ecosystem.", "The levels of ecological interactions within an ecosystem.", "The levels of energy flow within an ecosystem."],
    correctIndex: 3,
    explanation: "Trophic levels represent steps in the energy flow of an ecosystem, starting from producers up to various levels of consumers."
  },
  {
    text: "Which of the following statements is true regarding the urinary tubule in the excretory system?",
    options: ["The urinary tubule is responsible for the production of urine.", "The urinary tubule connects the kidneys to the bladder.", "The urinary tubule regulates the water and electrolyte balance in the body.", "The urinary tubule is the site of filtration of blood."],
    correctIndex: 0,
    explanation: "The urinary tubule (part of the nephron) processes the filtrate through reabsorption and secretion to produce the final urine."
  },
  {
    text: "Which of the following statements best describes courtship behaviors in animals?",
    options: [
      "Courtship behaviors are aggressive interactions between males competing for a female mate.",
      "Courtship behaviors involve displays and rituals performed by both males and females to attract a mate.",
      "Courtship behaviors are primarily performed by females to attract males for mating.",
      "Courtship behaviors are solely performed by males to establish dominance within a social group."
    ],
    correctIndex: 1,
    explanation: "Courtship involves specific rituals and displays used by animals to attract a mate and ensure successful reproduction."
  },
  {
    text: "Which of the following structures in the ear is responsible for transmitting sound vibrations to the auditory nerve?",
    options: ["Auditory canal", "Cochlea", "Eardrum", "Ossicles"],
    correctIndex: 1,
    explanation: "The cochlea converts mechanical sound vibrations into electrical nerve signals that travel to the auditory nerve."
  },
  {
    text: "Which of the following is a plant hormone responsible for promoting cell elongation and growth?",
    options: ["Gibberellins", "Abscisic acid", "Ethylene", "Cytokinins"],
    correctIndex: 0,
    explanation: "Gibberellins are primary plant hormones for stem elongation and overall plant growth."
  },
  {
    text: "Which of the following represents the correct hierarchical organization of life from the smallest to the largest scale?",
    options: [
      "Tissues, organs, cells, organisms, populations, communities, ecosystems.",
      "Organs, tissues, cells, organisms, populations, communities, ecosystems.",
      "Cells, tissues, organs, organisms, populations, communities, ecosystems.",
      "Cells, organs, tissues, organisms, populations, communities, ecosystems."
    ],
    correctIndex: 2,
    explanation: "Life is organized as: Cells → Tissues → Organs → Organisms → Populations → Communities → Ecosystems."
  },
  {
    text: "Metamorphosis is a biological process that involves",
    options: [
      "The change in form and structure during the life cycle of certain organisms.",
      "The transformation of an organism from an adult stage to a larval stage.",
      "The growth and development of an organism from a zygote to an adult.",
      "The regeneration of lost body parts in an organism."
    ],
    correctIndex: 0,
    explanation: "Metamorphosis involves significant physical changes in form, such as a caterpillar turning into a butterfly."
  },
  {
    text: "Which of the following best describes a natural habitat in ecology?",
    options: [
      "An area where organisms naturally live and interact with their surroundings.",
      "A controlled laboratory setting for ecological experiments.",
      "A human-created environment for wildlife conservation.",
      "A protected area for endangered species."
    ],
    correctIndex: 0,
    explanation: "A natural habitat is the specific environment where an organism naturally occurs and lives."
  },
  {
    text: "Which of the following is the correct classification of carbohydrates?",
    options: ["Micronutrient", "Phytonutrient", "Lipid", "Macronutrient"],
    correctIndex: 3,
    explanation: "Carbohydrates are macronutrients because the body requires them in large amounts for energy."
  },
  {
    text: "Which of the following statements is true regarding cell growth?",
    options: [
      "Cell growth involves an increase in the number of organelles within a cell.",
      "Cell growth is solely influenced by external factors.",
      "Cell growth occurs by cell division.",
      "Cell growth is a continuous process throughout the life of a cell."
    ],
    correctIndex: 2,
    explanation: "In biological contexts, tissue and organismal growth is achieved through the process of cell division (mitosis)."
  },
  {
    text: "Germination is the process in which a seed",
    options: ["Absorbs nutrients from the soil.", "Breaks dormancy and starts to grow.", "Begins to photosynthesize.", "Develops into a mature plant."],
    correctIndex: 1,
    explanation: "Germination begins when a seed takes in water and breaks its dormant state to begin growing into a seedling."
  },
  {
    text: "Which of the following best describes physiological variation in biology?",
    options: [
      "Variations in the genetic makeup of individuals within a species.",
      "Variations in the physiological processes and functions of organisms.",
      "Differences in physical characteristics and appearance within a population.",
      "Differences in behavior and social interactions among individuals."
    ],
    correctIndex: 1,
    explanation: "Physiological variation refers to differences in how internal systems (like metabolism or heartbeat) function among individuals."
  },
  {
    text: "Which of the following eye defects is caused by the inability of the eye to focus light on the retina?",
    options: ["Cataracts", "Myopia", "Glaucoma", "Astigmatism"],
    correctIndex: 1,
    explanation: "Myopia (nearsightedness) occurs when light focuses in front of the retina instead of directly on it."
  },
  {
    text: "Which of the following options best describes adaptation for survival in organisms?",
    options: [
      "Adaptation involves the development of new traits in response to changes in the environment.",
      "Adaptation refers to the ability of an organism to change its environment to better suit its needs.",
      "Adaptation is the inherited trait that increases an organism's chances of survival and reproduction in its environment.",
      "Adaptation is the process by which organisms acquire new characteristics during their lifetime."
    ],
    correctIndex: 2,
    explanation: "Adaptations are inherited characteristics that enhance survival and reproductive success in a specific environment."
  },
  {
    text: "Which of the following options correctly identifies excretory organs in animals?",
    options: ["Lungs, kidneys, and skin", "Heart, liver, and spleen", "Brain, spinal cord, and nerves", "Stomach, intestines, and bladder"],
    correctIndex: 0,
    explanation: "The lungs (CO2), kidneys (urea/urine), and skin (sweat) are the primary organs for removing metabolic waste."
  },
  {
    text: "Which of the following statements best describes pollination in plants?",
    options: [
      "Pollination is the process of releasing pollen into the air for dispersal.",
      "Pollination is the process of seed formation within a flower.",
      "Pollination is the process of transferring pollen from the anther to the stigma of a flower.",
      "Pollination is the process of transferring pollen from the stigma to the anther of a flower."
    ],
    correctIndex: 2,
    explanation: "Pollination is the transfer of pollen grains from the male anther to the female stigma."
  },
  {
    text: "Which of the following is a method of asexual reproduction in plants?",
    options: ["Fertilization", "Vegetative propagation", "Pollination", "Seed dispersal"],
    correctIndex: 1,
    explanation: "Vegetative propagation allows plants to reproduce using parts like stems or roots without needing seeds or gametes."
  },
  {
    text: "Which of the following molecules is the final electron acceptor in cellular respiration?",
    options: ["Glucose", "Oxygen", "Water", "Carbon dioxide"],
    correctIndex: 1,
    explanation: "Oxygen acts as the final electron acceptor in the electron transport chain to form water."
  },
  {
    text: "What is the primary function of the liver in the human body?",
    options: ["Regulation of blood pressure", "Regulation of body temperature", "Detoxification and metabolism of nutrients and drugs", "Production of hormones"],
    correctIndex: 2,
    explanation: "The liver filters toxins from the blood and manages the chemical levels and metabolism of nutrients."
  },
  {
    text: "Which of the following plant tissues is responsible for transporting water and nutrients from the roots to the rest of the plant?",
    options: ["Mesophyll", "Xylem", "Phloem", "Epidermis"],
    correctIndex: 1,
    explanation: "Xylem is the vascular tissue responsible for the upward conduction of water and minerals."
  },
  {
    text: "Which of the following mechanisms is responsible for providing support in plants?",
    options: ["Cell walls and turgor pressure", "Muscles and bones", "Endocrine system", "Exoskeleton"],
    correctIndex: 0,
    explanation: "Rigid cell walls and internal fluid pressure (turgor pressure) keep plant structures upright."
  },
  {
    text: "Which of the following characteristics distinguishes monocotyledonous plants from dicotyledonous plants?",
    options: ["Method of reproduction", "Presence of vascular bundles", "Number of cotyledons in the seed", "Same type of root system"],
    correctIndex: 2,
    explanation: "Monocots have one seed leaf (cotyledon), while dicots have two."
  },
  {
    text: "Which of the following is a primary source of pollution in aquatic ecosystems?",
    options: ["Air pollution", "Deforestation", "Soil erosion", "Industrial discharge"],
    correctIndex: 3,
    explanation: "Industrial discharge directly dumps chemicals and waste into water bodies, causing primary aquatic pollution."
  },
  {
    text: "Which of the following is an example of conserving resources in an ecosystem?",
    options: ["Implementing sustainable fishing practices.", "Excessive use of chemical fertilizers in agriculture.", "Introducing invasive species to an ecosystem.", "Cutting down trees for timber production."],
    correctIndex: 0,
    explanation: "Sustainable fishing ensures fish populations are not depleted, preserving the ecosystem's resource balance."
  },
  {
    text: "Viviparity refers to the reproductive strategy in which",
    options: [
      "Offspring develop and are nourished outside the female's body.",
      "Offspring are produced by external fertilization.",
      "Offspring are produced by internal fertilization.",
      "Offspring develop and are nourished inside the female's body."
    ],
    correctIndex: 3,
    explanation: "Viviparity involves giving birth to live young that developed inside the mother's body."
  },
  {
    text: "Which of the following describes the inheritance of traits from parents to offspring?",
    options: ["Natural selection", "Genetics", "Evolution", "Adaptation"],
    correctIndex: 1,
    explanation: "Genetics is the biological field focused on how traits and genes are passed down through generations."
  },
  {
    text: "Which of the following is a difference between plant and animal cells?",
    options: [
      "Plant cells have a cell membrane, while animal cells have a cell wall.",
      "Plant cells have a nucleus, while animal cells do not.",
      "Plant cells contain chloroplasts for photosynthesis, while animal cells do not.",
      "Plant cells have a central vacuole, while animal cells have no vacuoles."
    ],
    correctIndex: 2,
    explanation: "Chloroplasts are unique to plant cells (and some algae) for producing food via photosynthesis."
  },
  {
    text: "Which of the following factors primarily affects the distribution of organisms in an ecosystem?",
    options: ["Wind speed", "Temperature", "Day length", "Soil pH"],
    correctIndex: 1,
    explanation: "Temperature determines metabolic rates and survival, making it a primary factor in where organisms can live."
  },
  {
    text: "Which of the following statements is true regarding sex-linked traits?",
    options: [
      "Sex-linked traits are not influenced by hormonal factors.",
      "Sex-linked traits are located on the sex chromosomes.",
      "Sex-linked traits are more commonly observed in females.",
      "Sex-linked traits are inherited only from the mother."
    ],
    correctIndex: 1,
    explanation: "Sex-linked traits are governed by genes located specifically on the X or Y chromosomes."
  },
  {
    text: "Which of the following organs is primarily responsible for excretion in humans?",
    options: ["Kidneys", "Lungs", "Pancreas", "Liver"],
    correctIndex: 0,
    explanation: "The kidneys are the chief organs for filtering blood and excreting waste in the form of urine."
  },
  {
    text: "Which of the following is NOT a method of reproduction in animals?",
    options: ["Sexual reproduction", "Asexual reproduction", "Budding", "Sporulation"],
    correctIndex: 3,
    explanation: "Sporulation is typical of fungi and bacteria, not animals."
  },
  {
    text: "Which of the following statements about viruses is true?",
    options: [
      "Viruses possess a cellular structure.",
      "Viruses are living organisms.",
      "Viruses require a host cell to replicate.",
      "Viruses can reproduce outside of a host cell."
    ],
    correctIndex: 2,
    explanation: "Viruses are non-cellular obligate parasites; they cannot reproduce without a living host cell."
  },
  {
    text: "Which of the following characteristics is typical of the phylum Arthropoda?",
    options: ["Endoskeleton made of bones.", "Closed circulatory system.", "Presence of a segmented body.", "Radial symmetry."],
    correctIndex: 2,
    explanation: "Arthropods (insects, crustaceans, etc.) are defined by segmented bodies and jointed appendages."
  }
];

// --- HELPER: SHUFFLE LOGIC ---
const shuffleQuestions = (array) => {
  const shuffledQuestions = [...array].sort(() => Math.random() - 0.5);
  return shuffledQuestions.map(q => {
    const optionsWithMetadata = q.options.map((opt, idx) => ({
      text: opt,
      isCorrect: idx === q.correctIndex
    }));
    const shuffledOptions = optionsWithMetadata.sort(() => Math.random() - 0.5);
    const newCorrectIndex = shuffledOptions.findIndex(opt => opt.isCorrect);
    return {
      ...q,
      options: shuffledOptions.map(o => o.text),
      correctIndex: newCorrectIndex
    };
  });
};

export default function App() {
  // Auth State
  const [user, setUser] = useState(null);
  
  // App Navigation State
  const [view, setView] = useState('landing'); // 'landing', 'exam', 'results', 'admin'
  const [fullName, setFullName] = useState('');
  const [showNameError, setShowNameError] = useState(false);
  const [attempts, setAttempts] = useState(parseInt(localStorage.getItem('bb_attempts') || '0'));
  
  // Exam Execution State
  const [currentQuestionIdx, setCurrentQuestionIdx] = useState(0);
  const [answers, setAnswers] = useState({}); 
  const [timeLeft, setTimeLeft] = useState(1200); 
  const [isSubmitted, setIsSubmitted] = useState(false);
  const [sessionQuestions, setSessionQuestions] = useState([]);
  const [securityViolation, setSecurityViolation] = useState(false);

  // Admin State
  const [adminReports, setAdminReports] = useState([]);
  const [isLoadingReports, setIsLoadingReports] = useState(false);

  // 1. Firebase Auth Initialization
  useEffect(() => {
    const initAuth = async () => {
      if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
        await signInWithCustomToken(auth, __initial_auth_token);
      } else {
        await signInAnonymously(auth);
      }
    };
    initAuth();
    const unsubscribe = onAuthStateChanged(auth, (u) => {
      setUser(u);
    });
    return () => unsubscribe();
  }, []);

  // 2. Anti-Cheat: Tab Switching Detection
  useEffect(() => {
    const handleVisibilityChange = () => {
      if (document.visibilityState === 'hidden' && view === 'exam' && !isSubmitted) {
        setSecurityViolation(true);
        handleSubmit("Security Violation: User left the page");
      }
    };
    document.addEventListener("visibilitychange", handleVisibilityChange);
    return () => document.removeEventListener("visibilitychange", handleVisibilityChange);
  }, [view, isSubmitted]);

  // 3. Exam Timer Logic
  useEffect(() => {
    let timerId;
    if (view === 'exam' && !isSubmitted && timeLeft > 0) {
      timerId = setInterval(() => {
        setTimeLeft(prev => prev - 1);
      }, 1000);
    } else if (timeLeft === 0 && view === 'exam' && !isSubmitted) {
      handleSubmit("Time Expired");
    }
    return () => clearInterval(timerId);
  }, [view, isSubmitted, timeLeft]);

  // --- ACTIONS ---

  const startExam = () => {
    if (!fullName || fullName.trim().length < 3) {
      setShowNameError(true);
      return;
    }
    if (attempts >= 2) return; // Prevent start if max attempts reached

    const newAttempts = attempts + 1;
    setAttempts(newAttempts);
    localStorage.setItem('bb_attempts', newAttempts.toString());

    setSessionQuestions(shuffleQuestions(RAW_QUESTIONS));
    setView('exam');
    setCurrentQuestionIdx(0);
    setAnswers({});
    setTimeLeft(1200); 
    setIsSubmitted(false);
    setSecurityViolation(false);
    setShowNameError(false);
    window.scrollTo(0, 0);
  };

  const handleSubmit = async (reason = "Normal Completion") => {
    if (isSubmitted) return;
    setIsSubmitted(true);
    
    // Calculate final stats
    const totalQuestions = sessionQuestions.length || 40; 
    const correctCount = sessionQuestions.reduce((total, q, idx) => {
      return total + (answers[idx] === q.correctIndex ? 1 : 0);
    }, 0);
    const skippedCount = totalQuestions - Object.keys(answers).length;
    const finalScore = Math.round((correctCount / totalQuestions) * 100);

    // Sync to backend (Firestore)
    if (user) {
      try {
        await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'reports'), {
          candidateName: fullName,
          score: finalScore,
          correct: correctCount,
          wrong: totalQuestions - correctCount - skippedCount,
          skipped: skippedCount,
          attemptNumber: attempts + 1,
          durationUsed: 1200 - timeLeft,
          status: reason,
          timestamp: serverTimestamp(),
          uid: user.uid
        });
      } catch (e) {
        console.error("Failed to sync result:", e);
      }
    }

    setView('results');
    window.scrollTo(0, 0);
  };

  const handleOpenAdmin = async () => {
    const password = prompt("Admin Passkey:");
    if (password !== "admin123") {
      alert("Unauthorized access.");
      return;
    }
    setView('admin');
    setIsLoadingReports(true);
    try {
      const q = query(collection(db, 'artifacts', appId, 'public', 'data', 'reports'), orderBy('timestamp', 'desc'));
      const snapshot = await getDocs(q);
      const reports = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
      setAdminReports(reports);
    } catch (e) {
      console.error(e);
    } finally {
      setIsLoadingReports(false);
    }
  };

  const formatTime = (seconds) => {
    const m = Math.floor(seconds / 60).toString().padStart(2, '0');
    const s = (seconds % 60).toString().padStart(2, '0');
    return `${m}:${s}`;
  };

  // --- SUB-VIEWS ---

  if (view === 'results') {
    const totalQuestions = sessionQuestions.length; 
    const correctCount = sessionQuestions.reduce((total, q, idx) => {
      return total + (answers[idx] === q.correctIndex ? 1 : 0);
    }, 0);
    const finalScore = Math.round((correctCount / totalQuestions) * 100);

    return (
      <div className="min-h-screen bg-slate-50 py-12 px-4 font-sans">
        <div className="max-w-4xl mx-auto">
          {securityViolation && (
            <div className="bg-rose-600 text-white p-4 rounded-2xl mb-6 flex items-center gap-3 animate-pulse shadow-lg">
              <AlertTriangle className="shrink-0" />
              <p className="font-bold text-sm uppercase tracking-wider">Security Alert: Automatic Submission Triggered (Left Exam Screen)</p>
            </div>
          )}
          
          <div className="bg-white rounded-3xl p-8 border border-slate-200 shadow-xl text-center mb-8 relative overflow-hidden">
            <div className="absolute top-0 left-0 w-full h-2 bg-purple-600"></div>
            <h2 className="text-3xl font-bold text-slate-900 mb-2">Test Results for {fullName}</h2>
            <p className="text-purple-600 font-bold mb-8 uppercase tracking-widest text-sm">BEACON BRAINS TUTORIAL '26</p>
            
            <div className="grid grid-cols-1 md:grid-cols-3 gap-6 mb-10">
              <div className="bg-slate-50 rounded-2xl p-6 border border-slate-100">
                <Target className="w-6 h-6 text-purple-500 mx-auto mb-2" />
                <p className="text-slate-500 text-xs uppercase font-bold">Accuracy</p>
                <p className="text-3xl font-black text-slate-900">{correctCount} <span className="text-lg text-slate-400">/ {totalQuestions}</span></p>
              </div>
              <div className="bg-purple-700 rounded-2xl p-6 shadow-lg shadow-purple-100">
                <Trophy className="w-6 h-6 text-purple-200 mx-auto mb-2" />
                <p className="text-purple-100 text-xs uppercase font-bold">Final Grade</p>
                <p className="text-4xl font-black text-white">{finalScore}<span className="text-xl">/100</span></p>
              </div>
              <div className="bg-slate-50 rounded-2xl p-6 border border-slate-100">
                <Clock className="w-6 h-6 text-purple-500 mx-auto mb-2" />
                <p className="text-slate-500 text-xs uppercase font-bold">Time Used</p>
                <p className="text-3xl font-black text-slate-900">{formatTime(1200 - timeLeft)}</p>
              </div>
            </div>
            
            <div className="flex flex-col sm:flex-row gap-4 justify-center">
              <button onClick={() => setView('landing')} className="px-8 py-3 bg-slate-100 text-slate-700 rounded-xl font-bold hover:bg-slate-200 transition-all">Back to Home</button>
              {attempts < 2 && (
                <button onClick={startExam} className="px-8 py-3 bg-purple-600 text-white rounded-xl font-bold hover:bg-purple-700 transition-all shadow-lg shadow-purple-200">Try Final Attempt</button>
              )}
            </div>
          </div>

          <div className="space-y-6">
            <h3 className="text-xl font-bold text-slate-800 mb-4 flex items-center gap-2">
              <BookOpen className="w-5 h-5 text-purple-600" /> Correction & Detailed Review
            </h3>
            {sessionQuestions.map((q, idx) => {
              const userChoice = answers[idx];
              const isCorrect = userChoice === q.correctIndex;
              return (
                <div key={idx} className={`p-6 rounded-2xl bg-white border shadow-sm ${isCorrect ? 'border-emerald-200' : 'border-rose-200'}`}>
                  <div className="flex justify-between items-start gap-4 mb-4">
                    <p className="font-bold text-slate-900 text-lg leading-snug">{idx + 1}. {q.text}</p>
                    <div className={`flex items-center gap-1 text-[10px] font-black px-2 py-1 rounded-lg shrink-0 uppercase tracking-tighter ${isCorrect ? 'bg-emerald-100 text-emerald-700' : 'bg-rose-100 text-rose-700'}`}>
                      {isCorrect ? <><CheckCircle2 className="w-3 h-3" /> Correct</> : <><XCircle className="w-3 h-3" /> {userChoice === undefined ? 'Skipped' : 'Wrong'}</>}
                    </div>
                  </div>
                  <div className="grid grid-cols-1 md:grid-cols-2 gap-3">
                    {q.options.map((opt, oIdx) => {
                      const isCorrectOption = oIdx === q.correctIndex;
                      const isUserOption = oIdx === userChoice;
                      let btnClass = isCorrectOption ? "bg-emerald-600 border-emerald-600 text-white font-bold" : isUserOption && !isCorrect ? "bg-rose-600 border-rose-600 text-white font-bold" : "bg-slate-50 border-slate-100 text-slate-500";
                      return (
                        <div key={oIdx} className={`p-4 rounded-xl text-sm border flex items-center gap-3 ${btnClass}`}>
                          <div className={`w-6 h-6 rounded-full flex items-center justify-center text-[10px] font-bold shrink-0 ${isCorrectOption || (isUserOption && !isCorrect) ? 'bg-white/20' : 'bg-slate-200 text-slate-500'}`}>
                            {String.fromCharCode(65 + oIdx)}
                          </div>
                          <span className="flex-grow">{opt}</span>
                        </div>
                      );
                    })}
                  </div>
                  <div className="mt-5 p-4 bg-purple-50 rounded-xl border border-purple-100 flex gap-3 items-start">
                    <Info className="w-5 h-5 text-purple-600 shrink-0 mt-0.5" />
                    <div>
                      <p className="text-purple-900 font-bold text-sm mb-1 uppercase tracking-wider">Explanation</p>
                      <p className="text-slate-600 text-sm leading-relaxed">{q.explanation}</p>
                    </div>
                  </div>
                </div>
              );
            })}
          </div>
        </div>
      </div>
    );
  }

  if (view === 'exam') {
    const currentQ = sessionQuestions[currentQuestionIdx];
    return (
      <div className="min-h-screen bg-slate-50 flex flex-col font-sans">
        <div className="bg-white border-b border-slate-200 px-6 py-4 flex justify-between items-center sticky top-0 z-20">
          <div className="flex items-center gap-4">
            <h1 className="font-bold text-slate-900 leading-none hidden md:block">Biology Mock Test</h1>
            <div className="bg-purple-100 text-purple-700 text-[10px] px-2 py-1 rounded font-bold uppercase tracking-widest">{fullName}</div>
          </div>
          <div className={`flex items-center gap-2 px-4 py-2 rounded-xl font-mono text-xl font-bold border ${timeLeft < 120 ? 'bg-rose-50 text-rose-600 border-rose-200 animate-pulse' : 'bg-slate-900 text-white'}`}>
            <Timer className="w-5 h-5" /> {formatTime(timeLeft)}
          </div>
          <button onClick={() => handleSubmit()} className="bg-purple-700 text-white px-5 py-2 rounded-lg font-bold hover:bg-purple-800 shadow-md transition-all active:scale-95">
            Submit
          </button>
        </div>
        <div className="flex-grow flex flex-col lg:flex-row max-w-7xl mx-auto w-full p-4 gap-6">
          <div className="flex-grow bg-white rounded-3xl p-6 md:p-10 shadow-sm border border-slate-200 flex flex-col">
            <div className="mb-8">
              <span className="bg-purple-100 text-purple-700 font-bold text-xs px-2 py-1 rounded uppercase">Question {currentQuestionIdx + 1} of {sessionQuestions.length}</span>
              <h2 className="text-2xl md:text-3xl font-bold text-slate-900 mt-4 leading-tight">{currentQ?.text}</h2>
            </div>
            <div className="space-y-4 mb-10 flex-grow">
              {currentQ?.options.map((opt, idx) => (
                <button 
                  key={idx} 
                  onClick={() => setAnswers({ ...answers, [currentQuestionIdx]: idx })}
                  className={`w-full text-left p-5 rounded-2xl border-2 transition-all flex items-center gap-4 group ${answers[currentQuestionIdx] === idx ? 'border-purple-600 bg-purple-50 ring-4 ring-purple-50' : 'border-slate-100 hover:border-purple-200 hover:bg-slate-50 bg-white'}`}
                >
                  <div className={`w-6 h-6 rounded-full border-2 flex items-center justify-center ${answers[currentQuestionIdx] === idx ? 'border-purple-600 bg-purple-600' : 'border-slate-300'}`}>
                    {answers[currentQuestionIdx] === idx && <div className="w-2 h-2 bg-white rounded-full"></div>}
                  </div>
                  <span className={`text-lg ${answers[currentQuestionIdx] === idx ? 'font-bold text-purple-900' : 'text-slate-700'}`}>{opt}</span>
                </button>
              ))}
            </div>
            <div className="flex justify-between items-center pt-8 border-t border-slate-100">
              <button disabled={currentQuestionIdx === 0} onClick={() => setCurrentQuestionIdx(v => v - 1)} className="flex items-center gap-2 font-bold text-slate-500 disabled:opacity-20 transition-colors">
                <ArrowLeft className="w-5 h-5" /> Previous
              </button>
              <button disabled={currentQuestionIdx === sessionQuestions.length - 1} onClick={() => setCurrentQuestionIdx(v => v + 1)} className="flex items-center gap-2 font-bold text-purple-700 disabled:opacity-20 transition-colors">
                Next <ArrowRight className="w-5 h-5" />
              </button>
            </div>
          </div>
          <div className="w-full lg:w-80 bg-white rounded-3xl p-6 border border-slate-200 shadow-sm h-fit sticky top-24">
            <div className="flex items-center gap-2 mb-4 font-bold text-slate-800 border-b pb-3">
              <LayoutGrid className="w-5 h-5 text-purple-600" /> Palette
            </div>
            <div className="grid grid-cols-5 gap-2">
              {sessionQuestions.map((_, idx) => (
                <button key={idx} onClick={() => setCurrentQuestionIdx(idx)} className={`h-10 rounded-lg text-sm font-bold border transition-all ${currentQuestionIdx === idx ? 'bg-purple-600 text-white border-purple-600 shadow-md scale-110' : answers[idx] !== undefined ? 'bg-emerald-50 text-emerald-700 border-emerald-100' : 'bg-slate-50 text-slate-400 border-slate-100'}`}>
                  {idx + 1}
                </button>
              ))}
            </div>
            <div className="mt-6 p-4 bg-rose-50 border border-rose-100 rounded-xl">
              <p className="text-[10px] text-rose-700 font-bold uppercase mb-1 flex items-center gap-1"><Shield className="w-3 h-3" /> Security Policy</p>
              <p className="text-[10px] text-slate-500 leading-tight">Switching tabs or leaving this screen will result in automatic submission of your exam.</p>
            </div>
          </div>
        </div>
      </div>
    );
  }

  if (view === 'admin') {
    return (
      <div className="min-h-screen bg-slate-900 text-white p-6 font-sans">
        <div className="max-w-6xl mx-auto">
          <div className="flex justify-between items-center mb-10">
            <h2 className="text-3xl font-black flex items-center gap-4">
              <Shield className="text-purple-500 w-10 h-10" /> ADMIN DASHBOARD
            </h2>
            <button onClick={() => setView('landing')} className="bg-white/10 hover:bg-white/20 px-6 py-2 rounded-xl font-bold transition-all">Exit Admin</button>
          </div>
          
          <div className="bg-white/5 border border-white/10 rounded-3xl overflow-hidden backdrop-blur-xl">
            {isLoadingReports ? (
              <div className="p-20 text-center"><Sparkles className="animate-spin mx-auto mb-4" /> Loading records...</div>
            ) : (
              <div className="overflow-x-auto">
                <table className="w-full text-left">
                  <thead className="bg-white/10 text-purple-300 text-xs font-black uppercase tracking-widest">
                    <tr>
                      <th className="p-6">Candidate Name</th>
                      <th className="p-6 text-center">Score</th>
                      <th className="p-6 text-center">Attempt</th>
                      <th className="p-6">Status / Integrity</th>
                      <th className="p-6">Detailed Stats</th>
                      <th className="p-6">Time Finished</th>
                    </tr>
                  </thead>
                  <tbody className="divide-y divide-white/5">
                    {adminReports.length === 0 ? (
                      <tr><td colSpan="6" className="p-20 text-center opacity-40">No records found.</td></tr>
                    ) : (
                      adminReports.map((report) => (
                        <tr key={report.id} className="hover:bg-white/5 transition-colors">
                          <td className="p-6 font-bold text-lg">{report.candidateName}</td>
                          <td className="p-6 text-center">
                            <span className={`inline-block px-4 py-1 rounded-full text-xl font-black ${report.score >= 50 ? 'bg-emerald-500/20 text-emerald-400' : 'bg-rose-500/20 text-rose-400'}`}>
                              {report.score}%
                            </span>
                          </td>
                          <td className="p-6 text-center opacity-70">#{report.attemptNumber}</td>
                          <td className="p-6">
                            <span className={`px-3 py-1 rounded-lg text-[10px] font-black uppercase ${report.status.includes('Violation') ? 'bg-rose-600 text-white' : 'bg-emerald-600/30 text-emerald-400 border border-emerald-500/30'}`}>
                              {report.status}
                            </span>
                          </td>
                          <td className="p-6">
                            <div className="flex gap-2 text-[10px] font-bold">
                              <span className="text-emerald-400">✓ {report.correct}</span>
                              <span className="text-rose-400">✗ {report.wrong}</span>
                              <span className="text-slate-400">○ {report.skipped}</span>
                            </div>
                          </td>
                          <td className="p-6 text-xs opacity-50 font-mono">
                            {report.timestamp?.toDate().toLocaleString()}
                          </td>
                        </tr>
                      ))
                    )}
                  </tbody>
                </table>
              </div>
            )}
          </div>
        </div>
      </div>
    );
  }

  return (
    <div className="min-h-screen bg-[#310A31] font-sans text-white relative overflow-hidden">
      <div className="absolute top-[-10%] left-[-10%] w-[40%] h-[40%] bg-purple-600 rounded-full blur-[150px] opacity-20"></div>
      <div className="absolute bottom-[-10%] right-[-10%] w-[40%] h-[40%] bg-indigo-600 rounded-full blur-[150px] opacity-20"></div>

      <div className="relative max-w-6xl mx-auto px-6 py-12 md:py-20 flex flex-col items-center z-10">
        <div className="flex justify-between w-full mb-8">
          <div className="p-4 bg-white/10 backdrop-blur-md rounded-3xl border border-white/20 shadow-2xl animate-bounce">
            <Sparkles className="w-10 h-10 text-purple-300" />
          </div>
          <button onClick={handleOpenAdmin} className="opacity-10 hover:opacity-100 transition-all p-4"><Shield /></button>
        </div>
        
        <div className="text-center mb-12">
          <h1 className="text-4xl md:text-6xl font-black tracking-tight mb-4 leading-none uppercase">
            WELCOME TO BEACON BRAINS <br />
            <span className="text-purple-300">TUTORIAL '26</span>
          </h1>
          <p className="text-xl md:text-2xl text-purple-100 italic font-medium opacity-90 tracking-wide">
            "Lightening minds, Building futures"
          </p>
        </div>

        <div className="grid grid-cols-1 lg:grid-cols-2 gap-12 w-full items-center">
          <div className="order-2 lg:order-1 flex justify-center">
             <div className="bg-white/5 backdrop-blur-sm p-4 rounded-[2.5rem] border border-white/10 shadow-2xl relative group w-full max-w-md">
              <div className="flex items-center gap-2 mb-4 px-2">
                <Calendar className="w-5 h-5 text-purple-300" />
                <span className="text-xs font-bold tracking-widest uppercase opacity-70">Official Timetable</span>
              </div>
              <div className="relative rounded-2xl overflow-hidden shadow-2xl border border-white/20 bg-slate-800">
                <img 
                  src="https://api.screenshotmachine.com/?key=0c7d42&url=https://i.imgur.com/vHqQ9mK.jpg&device=desktop&dimension=1024x768" 
                  alt="Official Timetable"
                  className="w-full h-auto object-contain transition-transform duration-500 group-hover:scale-[1.02]"
                  onError={(e) => { e.target.src = "https://images.unsplash.com/photo-1506784983877-45594efa4cbe?q=80&w=2000"; }}
                />
              </div>
            </div>
          </div>

          <div className="order-1 lg:order-2">
            <div className="bg-white/10 backdrop-blur-md p-10 md:p-14 rounded-[3rem] border border-white/20 shadow-2xl relative overflow-hidden h-full flex flex-col justify-center">
              <div className="absolute top-0 right-0 p-8 opacity-10">
                <Dna className="w-24 h-24 text-white" />
              </div>
              
              <div className="relative z-10 text-center lg:text-left">
                <span className="inline-block bg-purple-500/30 border border-purple-400/30 px-4 py-1.5 rounded-full text-[12px] font-black uppercase tracking-[0.2em] mb-6 text-purple-200">
                  Mock Exam Live
                </span>
                <h3 className="text-4xl font-black mb-6 leading-tight">BIOLOGY MOCK TEST</h3>
                
                <div className="mb-8 space-y-4">
                  <div>
                    <label className={`block text-xs font-black uppercase tracking-widest mb-2 ${showNameError ? 'text-rose-400' : 'text-purple-300'}`}>
                      {showNameError ? 'Full Name Required to Proceed' : 'Enter Your Full Name'}
                    </label>
                    <div className="relative">
                      <User className="absolute left-4 top-1/2 -translate-y-1/2 text-white/40 w-5 h-5" />
                      <input 
                        type="text" 
                        value={fullName}
                        onChange={(e) => setFullName(e.target.value)}
                        placeholder="e.g. Ebuka Emmanuel"
                        className={`w-full bg-white/10 border p-5 pl-12 rounded-2xl outline-none transition-all font-bold text-lg ${showNameError ? 'border-rose-500 animate-shake bg-rose-500/10' : 'border-white/10 focus:border-purple-400 focus:bg-white/20'}`}
                      />
                    </div>
                  </div>
                </div>

                <div className="flex items-center gap-4 mb-10 bg-black/20 p-4 rounded-2xl border border-white/5">
                  <div className="text-center flex-grow border-r border-white/10">
                    <p className="text-[10px] uppercase font-bold text-slate-400">Attempts</p>
                    <p className="text-xl font-black">{attempts} / 2</p>
                  </div>
                  <div className="text-center flex-grow">
                    <p className="text-[10px] uppercase font-bold text-slate-400">Duration</p>
                    <p className="text-xl font-black">20 MIN</p>
                  </div>
                </div>

                {attempts >= 2 ? (
                  <div className="bg-rose-600/20 border border-rose-500/50 p-6 rounded-2xl text-center">
                    <p className="text-rose-300 font-bold mb-1">Max Attempts Reached</p>
                    <p className="text-xs text-rose-200/60">You have already taken this test twice. Please contact your tutor for further assistance.</p>
                  </div>
                ) : (
                  <button 
                    onClick={startExam} 
                    className="w-full group relative inline-flex items-center justify-center px-10 py-6 font-black text-xl text-[#310A31] transition-all duration-300 bg-white rounded-2xl hover:bg-purple-100 shadow-xl active:scale-95"
                  >
                    START EXAMINATION <ArrowRight className="ml-2 w-6 h-6 group-hover:translate-x-2 transition-transform" />
                  </button>
                )}
              </div>
            </div>
          </div>
        </div>

        <footer className="mt-20 pt-10 border-t border-white/5 w-full text-center text-purple-200/20 text-xs font-bold tracking-widest uppercase">
          Beacon Brains Tutorial '26 • "Lightening minds, Building futures"
        </footer>
      </div>
    </div>
  );
}
