# Decision Neuron Quest - Specification

## Overview
An interactive educational game that visualizes a neural network learning to decide "Should I eat a cookie right now?" Users can create training examples, adjust parameters, and watch the network learn through animated training iterations.

## Core Concept
A fully functional neural network with:
- **Input Layer**: 4 neurons (calories, price, review score, activity level)
- **Hidden Layer**: 3 neurons
- **Output Layer**: 1 neuron (decision: eat cookie or don't eat cookie)

## Features

### 1. Input Controls Panel

#### Cookie Parameters (Sliders)
- **Calories**: 0-1000 calories
  - Range slider with numeric display
  - Default: 300
  - Updates in real-time
  
- **Price**: $0-$10
  - Range slider with numeric display
  - Default: $3
  - Display as currency format
  
- **Review Score**: 0-5 stars
  - Range slider with numeric display
  - Default: 3.5
  - Step: 0.1

#### Activity Level (Bias Input)
- **Slider**: 0-100% activity intensity
  - 0% = "Completely sedentary all day"
  - 50% = "Moderate activity"
  - 100% = "Intense workout completed"
  - Shows descriptive text based on level
  - Default: 50%

#### Decision Label
- **Radio buttons or toggle**: 
  - "Should eat cookie" (target output: 1)
  - "Should NOT eat cookie" (target output: 0)
  - Default: "Should eat cookie"

#### Action Buttons
- **Add Training Example**: Saves current inputs + decision as a training point
- **Clear All Examples**: Removes all user-created training examples

### 2. Pre-loaded Training Dataset
Include 8-12 diverse training examples that demonstrate different scenarios:

**Example scenarios:**
1. High calories (800), expensive ($8), low rating (2.0), no workout (10%) → Don't eat
2. Low calories (150), cheap ($1), high rating (4.5), good workout (80%) → Eat
3. Medium calories (400), medium price ($4), medium rating (3.0), moderate activity (50%) → Don't eat
4. Low calories (200), expensive ($7), high rating (5.0), intense workout (95%) → Eat
5. High calories (900), cheap ($2), medium rating (3.5), no activity (5%) → Don't eat
6. Medium calories (350), cheap ($1.50), high rating (4.8), light activity (30%) → Eat
7. Very high calories (1000), medium price ($5), low rating (1.5), moderate workout (60%) → Don't eat
8. Low calories (180), medium price ($3.50), high rating (4.2), very active (90%) → Eat

Users can see, toggle on/off, or delete individual pre-loaded examples.

### 3. Neural Network Visualization

#### Network Structure
```
[Input Layer]    [Hidden Layer]    [Output Layer]
   
Calories ●━━━━━━━━━●
                  ╱ ╲ ╲
Price ●━━━━━━━━━━━●   ╲
              ╱ ╱ ╲ ╲  ╲
Review ●━━━━━━━━━●   ╲  ╲━━━━● Decision
            ╱ ╱ ╱ ╲  ╲ ╱
Activity ●━━━━━━━●    ╲╱
                  
(Bias) ●━━━━━━━━━━━━━━━━━╱

4 inputs → 3 hidden → 1 output
```

#### Visual Elements
- **Neurons**: Circles with labels
  - Input neurons: labeled (Calories, Price, Review, Activity)
  - Hidden neurons: labeled H1, H2, H3
  - Output neuron: labeled "Decision" with threshold (e.g., >0.5 = "Eat Cookie")
  
- **Connections/Weights**: Lines between neurons
  - Thickness: proportional to absolute weight value
  - Color: 
    - Positive weights: blue gradient (darker = stronger)
    - Negative weights: red gradient (darker = stronger)
  - Display actual weight value on hover
  
- **Neuron Activation**: Fill color intensity
  - Brightness proportional to activation value (0-1)
  - Show numeric activation value inside or near neuron

- **Bias Connections**: Dashed lines from bias node to hidden and output layers

#### Animation During Training
- When step/train button is pressed:
  - Pulse animation on neurons being activated
  - Weight lines animate (ripple effect from input to output)
  - Color/thickness changes smoothly as weights update
  - Numbers update with fade transition
  - Duration: ~800ms per step

### 4. Training Controls

#### Buttons
- **Step**: 
  - Advances one training iteration
  - Randomly selects one training example
  - Performs forward pass + backpropagation
  - Updates weights with animation
  - Shows which example was used
  - Updates loss/accuracy displays
  
- **Train**: 
  - Runs multiple steps automatically
  - Configurable speed (slider: 1-10 steps/second)
  - Can be paused mid-training
  - Shows progress (e.g., "Step 47/100")
  - Button toggles to "Pause" when running
  
- **Reset**:
  - Reinitializes all weights randomly
  - Clears training history/loss graph
  - Keeps training examples intact
  - Confirmation dialog: "Reset network weights?"

#### Training Parameters
- **Learning Rate**: Slider (0.001 - 0.5)
  - Default: 0.1
  - Shows current value
  
- **Training Steps** (for Train button): Input field
  - Default: 100
  - Range: 1-10000

### 5. Feedback & Metrics Panel

#### Real-time Display
- **Current Prediction**: 
  - Shows network's current decision for the input sliders
  - Large display: "EAT COOKIE 🍪" or "DON'T EAT 🚫"
  - Confidence percentage (how close to 0 or 1)
  
- **Loss/Error**: 
  - Current loss value
  - Color-coded: red (high) → yellow (medium) → green (low)
  
- **Accuracy**: 
  - Percentage of training examples correctly classified
  - Evaluated after each step
  
#### Loss/Accuracy Graph
- Line chart showing loss over training steps
- X-axis: training iteration
- Y-axis: loss value
- Optional second line for accuracy
- Updates in real-time during training
- Clears on reset

#### Training Example List
- Scrollable list showing all training examples (pre-loaded + user-created)
- Each entry shows:
  - Input values (calories, price, review, activity)
  - Correct label (eat/don't eat)
  - Current prediction ✓ or ✗
  - Toggle to include/exclude from training
  - Delete button (for user-created only)

### 6. Layout & Responsive Design

#### Desktop Layout (>1024px)
```
┌─────────────────────────────────────────────────┐
│  Decision Neuron Quest 🍪                       │
├──────────────┬──────────────────────────────────┤
│              │                                  │
│   Input      │    Neural Network Visualization  │
│   Controls   │    (large, centered)             │
│              │                                  │
│   - Sliders  │                                  │
│   - Decision │                                  │
│   - Buttons  │                                  │
│              │                                  │
├──────────────┼──────────────────────────────────┤
│   Training   │    Metrics & Feedback            │
│   Controls   │    - Current Prediction          │
│   - Step     │    - Loss/Accuracy               │
│   - Train    │    - Graph                       │
│   - Reset    │                                  │
│   - Params   │                                  │
├──────────────┴──────────────────────────────────┤
│   Training Examples List                        │
│   (scrollable, horizontal)                      │
└─────────────────────────────────────────────────┘
```

#### Mobile Layout (<768px)
- Stacked vertical layout
- Network visualization scales down but remains readable
- Input sliders full-width
- Collapsible sections for controls
- Sticky header with current prediction
- Training examples as vertical list

### 7. Technical Implementation

#### Neural Network Architecture
```javascript
class NeuralNetwork {
  constructor() {
    // Weights: input → hidden (4x3 matrix)
    this.weightsIH = randomMatrix(4, 3);
    
    // Weights: hidden → output (3x1 matrix)
    this.weightsHO = randomMatrix(3, 1);
    
    // Biases
    this.biasH = randomMatrix(3, 1);
    this.biasO = randomMatrix(1, 1);
    
    // Learning rate
    this.learningRate = 0.1;
  }
  
  // Forward propagation with sigmoid activation
  predict(inputs) { ... }
  
  // Backpropagation training
  train(inputs, target) { ... }
}
```

#### Input Normalization
- Calories: divide by 1000 (0-1 range)
- Price: divide by 10 (0-1 range)
- Review: divide by 5 (0-1 range)
- Activity: already 0-1 (percentage / 100)

#### Activation Function
- **Sigmoid**: σ(x) = 1 / (1 + e^(-x))
- Used for all neurons (hidden and output)

#### Loss Function
- **Mean Squared Error (MSE)**: (prediction - target)²

#### Weight Initialization
- Random values between -1 and 1
- Use Xavier/He initialization for better convergence

### 8. Visual Design

#### Color Scheme
- Background: Clean white or light gray (#f5f5f5)
- Primary: Blue (#3b82f6) for positive weights
- Secondary: Red (#ef4444) for negative weights
- Success: Green (#10b981) for correct predictions
- Warning: Orange (#f59e0b) for medium loss
- Text: Dark gray (#1f2937)

#### Typography
- Headers: Bold, sans-serif
- Body: Regular sans-serif (system fonts for performance)
- Numbers: Monospace for consistency

#### Animations
- Smooth transitions: 300-800ms
- Easing: ease-in-out
- Pulse effect for active neurons
- Ripple effect on weight lines during propagation
- Number updates: fade old → fade in new

### 9. Educational Features

#### Info Tooltips
- Hover over neurons to see activation values
- Hover over weights to see exact values
- Hover over terms (loss, learning rate) for explanations

#### Explanatory Text
- Brief intro explaining the concept
- "What's happening?" panel during training steps
- Shows which example is being used
- Explains forward pass → error calculation → backpropagation

#### Tutorial Mode (Optional)
- Step-by-step walkthrough on first visit
- Highlights each element as it's explained
- Can be skipped or reopened

### 10. Performance Considerations

- Use CSS transforms for animations (GPU accelerated)
- Debounce slider inputs to avoid excessive re-renders
- Canvas or SVG for network visualization (SVG preferred for responsiveness)
- Limit graph data points (e.g., max 500, then sample)
- RequestAnimationFrame for smooth training animations

### 11. Accessibility

- Keyboard navigation for all controls
- ARIA labels for screen readers
- Sufficient color contrast (WCAG AA)
- Focus indicators on interactive elements
- Alt text for visual elements
- Reduced motion option for users who prefer it

### 12. File Structure

```
decision-neuron-quest/
├── index.html          # Main HTML structure
├── styles.css          # All styling
├── script.js           # Main application logic
├── neural-network.js   # Neural network class
├── visualization.js    # SVG/Canvas rendering
└── README.md          # Usage instructions
```

### 13. Success Metrics

The application is successful when:
- Users can create and save training examples
- Neural network visibly learns and improves accuracy
- Weight visualizations update smoothly during training
- Network can correctly classify most training examples after sufficient training
- Interface is responsive and works on mobile devices
- Animations are smooth (60fps) on modern devices

### 14. Future Enhancements (Out of Scope for V1)

- Export/import trained models
- Compare different network architectures
- Multiple activation functions
- Batch training vs single example
- Visualization of decision boundary
- Sound effects for training milestones
- Leaderboard for best-trained networks

## Development Notes

- Start with a working neural network before adding visualization
- Test with the pre-loaded examples to ensure training works
- Implement step-by-step training first, then add auto-train
- Use console logging extensively during development
- Validate input normalization carefully
- Test edge cases (all same examples, contradictory examples)
