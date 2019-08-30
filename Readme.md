# Airborne particle size distribution (PSD) data processing
Version: Beta_1.1  
Created on 26-Aug-2019    
Authur: RicC137
Email: y.x@u.nus.edu
## Cautions
**YOU MUST IMPORT DATA BEFORE RUNNING THIS CODE**   
* Import data type: Numeric matrix   
* Excluded rows with: Unimportant cells  
* An exercise can be dowload in [test_demo]()
```matlab
%% Load spreadsheet
rawmat = input('The filename occoured in Workspace window: ');
%% Data processing

% Formating
rawmat_size = size(rawmat);
set_rownum = input('Enter the row number in each batch: ');
margin = input('Enter the tolerant offset(1.0-0) for peak\n (e.g. 0.95 means 5% offset; 0.9 means 10% offset): ');
if mod(rawmat_size(1), set_rownum) == 0 
    cell_num = rawmat_size(1) / set_rownum;
    msg = ['Total ', num2str(cell_num), ' cells are being calculated, please wait...'];
    disp(msg)
    rowDist = zeros(1, cell_num) + set_rownum;
    cell = mat2cell(rawmat, rowDist); % equally divide raw matrix
    % (celldisp(cell)) this line for checking the cells
    
    % Cell manipulation
    spoilx = [];
    peak_x = [];
    peak_y = [];
    peak_time = [];
    peak_datum = [];
    sequence = [];
    max_peak_x = [];
    max_peak_y = [];
    max_peak_time = [];
    max_peak_datum = [];
    max_sequence = [];
    for i = 1:cell_num
        if cell{i,1}(1,3) > cell{i,1}(30,3) % sort cell in sequence of mass ascending
            sort_cell = flip(cell{i,1});
        else
            sort_cell = cell{i,1};
        end
        
        % Find peaks
        x = sort_cell(:,3);
        y = sort_cell(:,4);
        if all(diff(x)>0) % check monotonicity
            [pks, locs] = findpeaks(y, x); 
            peak_x = [peak_x; locs];
            peak_y = [peak_y; pks];
            
            [m, n] = find(y == pks'); % corresponding time & datum#
            datum = sort_cell(m, 1);
            peak_datum = [peak_datum; datum];
            time = sort_cell(m, 2);
            peak_time = [peak_time; time];
            
            count = size(pks);
            i_arr = zeros(count(1),1)+i;
            sequence = [sequence; i_arr];
            
            % Find max of peaks
            [o,p] = find(y == max(pks));
            if y(o-1) < y(o)*margin && y(o+1) < y(o)*margin % left & right data both < peak*margin
                max_peak_x = [max_peak_x; max(locs)];
                max_peak_y = [max_peak_y; max(pks)];
                
            elseif y(o-1) > y(o)*margin && y(o+1) < y(o)*margin % left > peak*margin, right < peak*margin
                max_peak_x = [max_peak_x; mean([x(o-1),x(o)])];
                max_peak_y = [max_peak_y; mean([y(o-1),y(o)])];
            
            elseif y(o-1) < y(o)*margin && y(o+1) > y(o)*margin % left < peak*margin, right > peak*margin
                max_peak_x = [max_peak_x; mean([x(o),x(o+1)])];
                max_peak_y = [max_peak_y; mean([y(o),y(o+1)])];
                
            else % left & right data both > peak*margin
                max_peak_x = [max_peak_x; mean([x(o-1), x(o), x(o+1)])];
                max_peak_y = [max_peak_y; mean(y(o-1), y(o), y(o+1))];
            end
            
            max_datum = sort_cell(o, 1);
            max_peak_datum = [max_peak_datum; max_datum];
            max_time = sort_cell(o, 2);
            max_peak_time = [max_peak_time; max_time];
            
            max_sequence = [max_sequence; i];
        else % spoiled data will be replaced with 0
            spoilx = [spoilx, i];
            peak_x = 0;
            peak_y = 0;
            peak_datum = 0;
            peak_time = 0;
            max_peak_x = 0;
            max_peak_y = 0;
            max_peak_datum = 0;
            max_peak_time = 0;
            sequence = [sequence; i];
            max_sequence = [max_sequence; i];
        end
    end
    peak_tab = array2table([sequence, peak_datum, peak_time, peak_x,...
        peak_y], 'VariableNames',{'Set','Datum','Time','Massfg',...
        'Density'});
    
    max_peak_tab = array2table([max_sequence, max_peak_datum,...
        max_peak_time, max_peak_x, max_peak_y], 'VariableNames',...
        {'Set','Datum','Time','Max_Massfg','Max_Density'});
    msg3 = ['Mass is not stricly incresing, error occured in set: ',...
        num2str(spoilx)];
    disp(msg3)
else
    msg2 = 'Error in sorting data, check your data frame';
    disp(msg2)
end
%% Export spreadsheet
filename = input('Export result as: ','s');
writetable(peak_tab, filename, 'FileType', 'spreadsheet', 'Sheet', 1);
writetable(max_peak_tab, filename, 'FileType', 'spreadsheet', 'Sheet', 2);
```
## Update (foreshow)
Beta_2.0 will envolve:
* User friendly GUI operation pannel
* Multiple files processing function
* Enhancement in stability
## Support
If you enjoy the script, please **star** my work
Your support is my best motivation
