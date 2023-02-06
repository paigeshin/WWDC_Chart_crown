```swift
import SwiftUI
import Charts

struct ContentView: View {
    var body: some View {
        ProductivityChart()
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}

struct Todo {
    let title: String
    var complete: Bool = false
    var completedDate: Date?
}

struct CharData {
    
    struct DataElement: Identifiable, Equatable {
        let id: String = UUID().uuidString
        let date: Date
        let itemsComplete: Double
    }
    
    static func createData(_ items: [Todo]) -> [DataElement] {
        return Dictionary(grouping: items, by: \.completedDate)
            .compactMap {
                guard let date = $0 else { return nil }
                return DataElement(date: date, itemsComplete: Double($1.count))
            }
            .sorted {
                $0.date < $1.date
            }
    }
    
}

extension Date {
    
    func updatedHour(_ value: Int) -> Date {
        return Calendar.current.date(bySettingHour: value, minute: 0, second: 0, of: self) ?? .now
    }
    
}

struct ProductivityChart: View {
    
    let data: [CharData.DataElement] = [
        CharData.DataElement(date: Date().updatedHour(8), itemsComplete: Double.random(in: 0...10)),
        CharData.DataElement(date: Date().updatedHour(9), itemsComplete: Double.random(in: 0...10)),
        CharData.DataElement(date: Date().updatedHour(10), itemsComplete: Double.random(in: 0...10)),
        CharData.DataElement(date: Date().updatedHour(11), itemsComplete: Double.random(in: 0...10)),
        CharData.DataElement(date: Date().updatedHour(12), itemsComplete: Double.random(in: 0...10)),
        CharData.DataElement(date: Date().updatedHour(13), itemsComplete: Double.random(in: 0...10)),
        CharData.DataElement(date: Date().updatedHour(14), itemsComplete: Double.random(in: 0...10)),
        CharData.DataElement(date: Date().updatedHour(15), itemsComplete: Double.random(in: 0...10)),
        CharData.DataElement(date: Date().updatedHour(16), itemsComplete: Double.random(in: 0...10)),
        CharData.DataElement(date: Date().updatedHour(17), itemsComplete: Double.random(in: 0...10)),
        CharData.DataElement(date: Date().updatedHour(18), itemsComplete: Double.random(in: 0...10)),
        CharData.DataElement(date: Date().updatedHour(19), itemsComplete: Double.random(in: 0...10)),
    ]
    
    // MARK: - PROPERTIES FOR DIGITAL CROWN
    /// The index of the highlighted chart value. This is used for crown scrolling.
    @State var highlightDateIndex: Int = 0
     
    /// The current offset of the crown while it's rotating. This is set from
    /// the value in the DigitalCrownEvent and used to show an intermidate
    /// (between detents) chart value in the view
    @State var crownOffset: Double = 0.0
    @State var isCrownIdle = true
    
    @State var crownPositionOpacity: CGFloat = 0.2
    
    // Display the current value while scrolling
    @State var chartDataRange = (0...6)
    private var chartData: [CharData.DataElement] {
        Array(self.data[self.chartDataRange.clamped(to: (0...self.data.count - 1))])
    }
    
    func isLastDataPoint(_ item: CharData.DataElement) -> Bool {
        if item == self.data[self.data.count - 1] {
            return true
        }
        return false
    }
    
    var body: some View {
        
        Chart(self.data) { dataPoint in
            BarMark(
                x: .value("Date", dataPoint.date),
                y: .value("Completed", dataPoint.itemsComplete)
            )
            .foregroundStyle(Color.accentColor)
            // MARK: DISPLATY Digital Crown Value
            .annotation(
                position: self.isLastDataPoint(dataPoint) ? .topLeading : .topTrailing,
                spacing: 0,
                content: {
                    Text("\(dataPoint.itemsComplete, specifier: "%.0f")")
                        .foregroundStyle(
                            dataPoint.date == self.data[self.highlightDateIndex].date ? Color.yellow : Color.clear
                        )
                })
         
            // MARK: RULE MARK FOR DIGITAL CROWN
            RuleMark(x: .value("Date", self.data[self.highlightDateIndex].date))
                .foregroundStyle(Color.yellow.opacity(self.crownPositionOpacity))
            
        }
        // MARK: DIGITAL CROWN
        .focusable()
        .digitalCrownRotation(
            detent: self.$highlightDateIndex,
            from: 0,
            through: self.data.count - 1,
            by: 1,
            sensitivity: .medium
        ) { crownEvent in
            self.isCrownIdle = false
            self.crownOffset = crownEvent.offset
        } onIdle: {
            self.isCrownIdle = true
        }
        // MARK: DIGITAL CROWN ANIMATION
        .onChange(of: self.isCrownIdle, perform: { newValue in
            withAnimation(newValue ? .easeOut: .easeIn) {
                self.crownPositionOpacity = newValue ? 0 : 1.0
            }
        })
        // MARK: Display the current value while scrolling
        .onChange(of: self.highlightDateIndex, perform: { newValue in
            withAnimation {
//                self.updateChartDataRange()
            }
        })
        // MARK: CUSTOMIZING X AXIS
        .chartXAxis {
            AxisMarks { _ in
                // Various Formats Are Ready. Must Match DataType `x: .value("Date", dataPoint.date),`
                AxisValueLabel(format: .dateTime)
            }
        }
        // MARK: CUSTOMIZING Y AXIS
        .chartYAxis {
            
            AxisMarks { value in
                AxisGridLine(
                    stroke: StrokeStyle(lineWidth: 0.5)
                )
                .foregroundStyle(.gray)
                if value.index < (value.count - 1) {
                    // Various Formats Are Ready. Must Match DataType `y: .value("Completed", dataPoint.itemsComplete)`
                    AxisValueLabel {
                        Text("\(value.count)")
                    }
                }
            }
            
        }

        .navigationTitle("Productivity")
        
    }
    
}

```
