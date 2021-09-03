# DTW_ForSolarWindEvaluation
# The following is the DTW script used in the work of Samara et al., 2021 (in prep), 
# for the evaluation of modeled solar wind time series. It was adapted from
# https://nipunbatra.github.io/blog/ml/2014/05/01/dtw.html 
# ---------------------------------------------------------
# ---------------------------------------------------------

# Let's take two random time series
series1=[1, 4, 7, 8, 10, 9, 6, 5, 2, 3, 3]
series2=[1, 1, 4, 7, 8, 10, 9, 6, 5, 2, 3]

cost = np.zeros((len(series1), len(series2))) 
DTW = np.ones((len(series1)+1, len(series2)+1)) 
DTW = DTW*np.infty 

# Time window
w = np.max([300, abs(len(series1)-len(series2))]) 

DTW[0,0] = cost[0,0] 
        
for i in range(1, len(series1)+1):
    for j in range(np.max([1, i-w]), np.min([len(series2)+1, i+w])):   
        cost[i-1,j-1] = abs(series2[j-1]-series1[i-1])
        DTW[i,j] = min(DTW[i-1, j-1], DTW[i-1, j], DTW[i, j-1]) + cost[i-1,j-1] 
        
DTW = DTW[1:,1:] 
        
print("The DTW cost is ", DTW[len(series1)-1, len(series2)-1])
print(DTW)

# Plot the DTW cost matrix, the warping path and the alignments between the points

def distance_distance_plot(DTW):
    im = plt.imshow(DTW, interpolation='nearest', cmap='Greens') 
    #plt.gca().invert_yaxis()
    plt.xlabel("series 2", fontsize=20)
    plt.ylabel("series 1", fontsize=20)
    plt.grid()
    plt.colorbar();
    #plt.clim(0,45)
distance_distance_plot(DTW)

path = [[len(series2)-1, len(series1)-1]]
i = len(series1)-1
j = len(series2)-1
while i>1 or j>1:
    if i==0:
        j = j - 1
    elif j==0:
        i = i - 1
    else:
        if DTW[i-1, j] == min(DTW[i-1, j-1], DTW[i-1, j], DTW[i, j-1]):
            i = i - 1
        elif DTW[i, j-1] == min(DTW[i-1, j-1], DTW[i-1, j], DTW[i, j-1]):
            j = j-1
        else:
            i = i - 1
            j = j - 1
    path.append([j, i])
path.append([0,0])    

path_x = [point[0] for point in path]
path_y = [point[1] for point in path]
distance_distance_plot(DTW)
plt.plot(path_x, path_y, linewidth=5.0);

def path_DTW(series2, series1, DTW, cost):
    path = [[len(series2)-1, len(series1)-1]]
    DTW_new = 0
    i = len(series1)-1
    j = len(series2)-1
    while i>1 or j>1:
        if i==0:
            j = j - 1
        elif j==0:
            i = i - 1
        else:
            if DTW[i-1, j] == min(DTW[i-1, j-1], DTW[i-1, j], DTW[i, j-1]):
                i = i - 1
            elif DTW[i, j-1] == min(DTW[i-1, j-1], DTW[i-1, j], DTW[i, j-1]):
                j = j-1
            else:
                i = i - 1
                j= j- 1
        path.append([j, i])
    path.append([0,0])
    for [series1, series2] in path:
        DTW_new = DTW_new +cost[series2, series1]
    return path, DTW_new

plt.figure(figsize=(15,5))
plt.plot(series1, 'b-', label = 'series 1', linewidth=7)
plt.plot(series2, 'r-' ,label='series 2', linewidth=3)

plt.legend(fontsize=22);
paths = path_DTW(series2, series1, DTW, cost)[0]
for [map_x, map_y] in paths:
    plt.plot([map_x, map_y], [series2[map_x], series1[map_y]], 'g', linewidth=3)



